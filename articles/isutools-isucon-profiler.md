---
title: "ISUCON14で50万点を取るために行ったこと"
emoji: "🚕"
type: "tech"
topics: ["isucon", "go", "mysql", "performance"]
published: false
---

ISUCON14のGo実装を改善し、95,656点から569,078点まで上げました。

この記事では、50万点を超えるまでに実際に行った修正を順番にまとめます。ベンチマーカーのcorrectnessを通過した結果だけを採用しています。

## スコアの推移

主要な結果だけを抜き出すと、次のようになりました。

| 主な変更 | score |
|---|---:|
| 計測とサービスを復旧 | 95,656 |
| rolling Hungarian | 102,260 |
| nearbyの位置とavailabilityを整理 | 210,955 |
| notification response cache | 236,006 |
| matcherを100ms周期に変更 | 285,746 |
| coordinateのtransactionを短縮 | 309,372 |
| ride statusをmemoryから返す | 359,732 |
| matcherを10ms周期に変更 | 437,852 |
| paymentのtransaction短縮と100ms grace | 542,441 |
| matchingとnearbyの再公開を分離 | 567,276 |
| ride履歴のN+1をcache化 | **569,078** |

## 最初にindexとN+1を修正しました

`chairs`、`chair_locations`、`rides`、`ride_statuses`、`coupons`に複合indexを追加しました。

`GET /api/app/nearby-chairs`とowner系APIは、chairごとにSQLを実行していたため、まとめて取得するSQLへ変更しました。

この段階で、一度だけ遅いSQLより、notificationから何十万回も呼ばれる短いSQLの方が大きな負荷になることが分かりました。

## matcherを1件ずつの処理から一括割り当てに変えました

初期matcherは500msごとに1件しか割り当てませんでした。未割当rideと空いているchairが増えても、約2件/秒が上限です。

そこで、未割当rideとfree chairをまとめて取得し、距離とchairの速度をcostにしたHungarian法で一括割り当てするようにしました。

ISURIDEには離れた2地域があります。遠い地域のchairを無理に割り当てず、長く待っているrideだけをfallback対象にしました。

現在の実装は、10msごとに現在の候補を解くrolling Hungarianです。一度決めた割り当ての解除や、busy chairの将来位置の予測は行っていません。

## 割り当てをGIFで比較しました

左が95,656点時の実装、右が567,276点の実装です。

![ISUCON14の初期実装と最終実装の割り当て比較](https://ekusiadadus.com/images/isucon14/matching-before-after-567276.gif)

| 指標 | 初期版 | 567,276点版 |
|---|---:|---:|
| assignments | 1,810 | 10,760 |
| 距離400以上の地域間割り当て | 491 | 0 |
| pickupまでの平均距離 | 211.3 | 13.25 |

初期のartifactにはassignment eventがなかったため、左側は95,656点時のcommitへ計測だけを追加して再実行しています。右側は567,276点runの実データです。

## notificationのDB pollingを減らしました

appとchairはnotification APIを高頻度で呼びます。同じride statusを毎回DBから組み立てていたため、DB connection poolを使い続けていました。

rideごとにgenerationを持つresponse cacheを追加し、statusやassignmentが変わったときだけ更新します。未送信のnotificationが残っている場合はcacheせず、先にqueueを処理します。

状態が変わらない場合はJSON APIのままlong pollingするようにしました。

| notification | 変更前 | 変更後の比較run |
|---|---:|---:|
| app | 約155万回 | 約14.5万回 |
| chair | 約37.9万回 | 約5.6万回 |

SSEには変更していません。

## coordinateとride statusをmemoryに移しました

chairは`POST /api/chair/coordinate`の応答が返るまで次の移動を始めません。このAPIが遅いと、rideの完了数も増えません。

次の変更を行いました。

- INSERT直後に同じ座標をSELECTする処理を削除
- 通常の位置更新から長いtransactionを削除
- 最新座標をmemoryに保持
- 最新ride statusをmemoryに保持
- initialize時にcacheを作り直す

coordinateは107,643 calls、平均73.6msから、191,528 calls、平均2.2msになりました。

ownerのchair総走行距離も、リクエストごとに全座標から計算するのをやめました。initialize時に総距離を作り、その後はcoordinate更新時に移動距離を加算します。

## matcherの周期を10msにしました

matcherのSQLとcacheを整理してから、実行周期を100msから10msへ短縮しました。

この変更で389,614点から437,852点になりました。先にmatcher自体を軽くしていたため、周期を短くしてもDB負荷を増やしすぎずに済みました。

## paymentをDB transactionの外へ出しました

rideのevaluationでは、外部payment APIをDB transaction内から呼んでいました。paymentの応答待ち中もconnectionとrow lockを保持します。

修正後は、ride IDを`Idempotency-Key`に設定してpaymentを先に実行し、成功した後だけ短いtransactionを開きます。

ただし、処理が速くなったことで、serverがchairを再公開するタイミングがbenchmark clientの評価完了より先になる問題が出ました。

最初はchairの再利用を100ms遅らせ、542,441点になりました。その後、matcherにはすぐ再公開し、`GET /api/app/nearby-chairs`に対してだけclientの次回requestまで非表示にしました。これで567,276点になりました。

## 最後にride履歴のN+1を削除しました

567,276点のレポートでは、`GET /api/app/rides`から次のSQLがそれぞれ90,348回呼ばれていました。

```sql
SELECT * FROM chairs WHERE id = ?;
SELECT * FROM owners WHERE id = ?;
```

既存のchair cacheを再利用し、ownerもIDでcacheするようにしました。

| 指標 | 変更前 | 変更後 |
|---|---:|---:|
| score | 567,276 | **569,078** |
| chairのSELECT | 90,348回 | 11回 |
| ownerのSELECT | 90,348回 | 3回 |
| `GET /api/app/rides` p95 | 33.55ms | 16.78ms |
| DB pool wait合計 | 922.15秒 | 575.28秒 |

スコア差は小さいですが、対象SQLとAPIのp95は減りました。次のボトルネックが別の場所へ移ったと判断しています。

## 採用しなかった変更

50万点を超えても、correctness errorが出たrunは採用していません。

| 変更 | score | 結果 |
|---|---:|---|
| payment直後にchairを即時公開 | 591,636 | code 30が205件 |
| nearby応答まで広くlock | 561,815 | code 30、31が発生 |
| matchingとnearbyを両方即時公開 | 637,737 | code 30が13件 |
| DB poolを100から200へ変更 | 546,855 | correctnessはpass、スコアが低下 |

DB poolは100に戻しました。最も高いraw scoreではなく、correctnessを保った569,078点を最終結果にしています。

## まとめ

50万点を超えるまでに効果が大きかったのは、次の変更でした。

- matcherを1件ずつから一括割り当てに変更
- 2地域をまたぐ長距離割り当てを停止
- notificationとride statusのDBアクセスをcacheとlong pollingで削減
- coordinateのtransactionを短縮して最新状態をmemoryに保持
- matcher周期を10msへ短縮
- payment待ちをDB transactionの外へ移動
- matchingとnearbyでchairの再公開タイミングを分離

計測には[isutools](https://github.com/ekusiadadus/isutools)を使いました。全runのartifact、失敗したcommit、詳細な時系列は[本サイトの記事](https://ekusiadadus.com/ja/blog/isucon14-542k-with-isutools)に残しています。
