---
title: "ISUCON13を12,136点から445,684点まで改善した"
emoji: "🔥"
type: "tech"
topics: ["isucon", "go", "mysql", "pprof", "performance"]
published: false
---

WSL2上に構築したISUCON13のGo参考実装を、公式ベンチで **12,136点から445,684点** まで改善しました。最終runは`pass=true`で、ベンチマーカーの最終チェックも成功しています。

計測には[isutools](https://github.com/ekusiadadus/isutools)を使いました。この記事では、再現に必要なベンチ境界、SQL/HTTPの見方、pprofとPGO、採用しなかった変更を技術寄りにまとめます。

## 結果

| 状態 | score | pass |
|---|---:|---|
| WSL構築直後 | 12,706 | true |
| 計測条件を固定したbaseline | 12,136 | true |
| 最終採用版 | **445,684** | true |
| `make bench`再現run | 441,439 | true |
| pprof UI修正後の確認run | 433,468 | true |

最終採用版は比較可能baselineの36.72倍です。ただし、複数の変更を含むため、単一施策の効果が36.72倍だったとは扱いません。ベンチの揺れもあるので、各runのscore、pass、git revision、計測artifactを一緒に保存しました。

環境は次のとおりです。

- Windows 11 / WSL2
- Ubuntu 22.04.3 LTS
- [ISUCON13公式リポジトリ](https://github.com/isucon/isucon13)のGo参考実装
- nginx / MySQL / PowerDNS / isupipe-go
- `./bench run --dns-port 1053 --enable-ssl`
- 構築手順: [matsuu/wsl-isucon / isucon13](https://github.com/matsuu/wsl-isucon/tree/main/isucon13)

## 計測の開始と終了を固定する

最初に、毎回同じ順序でベンチと保存が動くようにしました。

```text
POST /reset
  -> 公式benchmark
  -> /tmp/result.jsonのscore/passを型検査
  -> POST /save?score=<score>&pass=<pass>
  -> Windows共有directoryへstage
  -> scpで制御PCへ取得
```

`/reset`の`X-Isutools-Run-Id`を、公式結果と計測snapshotの対応付けに使います。

最適化後は1分間のaccess logだけで約47万行になりました。明示的な`/collect`は固定2秒budgetを超えることがあったため、`/save`が終了境界を確定し、凍結したgenerationをdrainする経路にしました。

保存形式は次のようになります。

```text
official-benchmark-<time>-<run-id>.json
<time>_<seq>_gen<generation>_<rev>_score<score>.json
<time>_<seq>_gen<generation>_<rev>_score<score>.html
cpu_<capture-id>.pprof
cpu_<capture-id>.meta.json
```

`generation`はscoreの世代ではなく、reset前後の計測値を混ぜないための内部番号です。runの対応付けには`run_id`を使います。

## SQLは「一番遅い1回」より累計時間を見る

最初に見たのは、単発で最も遅いSQLではなく、実行回数と累計時間です。

livecomments、reactions、reports、tags、reservation、PowerDNS recordsなどへ、実際のfilter・join・orderに合わせてindexを追加しました。MySQL接続poolは50へ広げ、`interpolateParams=true`でprepared statementの往復も減らしました。

index追加後は次を確認しています。

- queryの実行回数と累計時間
- `EXPLAIN`の`key`と`rows`
- `Using filesort` / `Using temporary`
- 公式scoreと`pass`

`possible_keys`に候補が出ただけでは採用しません。optimizerが別のindexを選び、全体が遅くなることがあるためです。

## N+1とDB往復をmemory stateへ寄せる

user/theme/icon、livestream/tag、comment/reaction/report、NG word、統計、予約枠をinitialize時に構築し、その後はmemoryから参照できるようにしました。

1回数百µsのSQLでも、数万回呼ばれるとDB接続を長時間占有します。N+1を一括SQLまたはmemory集計に置き換え、read APIの不要なtransactionも外しました。

ただし、予約枠をmemoryだけで更新する案は採用しませんでした。局所処理は軽くなりましたが、DNS attackerの並列度が変わり、総合scoreが417,657へ低下したためです。

## icon GETの大半を304にする

icon hashをETagに使い、`If-None-Match`が一致する場合は304を返しました。

最終snapshotでは、icon GET 281,999件のうち281,642件が304でした。更新側はDELETE + INSERTではなく、`INSERT ... ON DUPLICATE KEY UPDATE`へ変更しています。

## pprofでCPUを使っているコードを確認する

CPU profileでは、JSON indentだけで累計21.36秒を消費していることが分かりました。Echoをproduction modeにし、API responseをcompact JSONへ変更しました。

高CPUだからpprofで解析できない、ということではありません。CPU profileはCPUを消費している場所を調べるためのものです。重要なのは、代表的な負荷区間を採取し、capture時binaryと解析binaryが一致していることです。

今回の解析では次を確認しました。

- benchmark区間全体のCPU profile
- binary SHA-256一致: `verified`
- CPU flame graph: `ready`
- flame graph: 2,048 nodes
- selected source root外のpathは`(redacted)`

レポート全体が`partial`でも、CPU解析が失敗したとは限りません。今回は外部依存のsource path redactionと、block/heap/mutex flameが`unsupported`であることが主な理由です。CPU flameは生成されています。

検証用ビルドでは、レポート上部に次のリンクを追加しました。

- `行解析結果`: verified analysisのfunction / file / lineへ移動
- `CPU pprofフレームグラフ`: `ready`なCPU flameを展開して移動

解析が未publishならProfilesへフォールバックします。保存済みHTMLはimmutableなので、新rendererで見るときはRuns一覧の`current UI`を使います。

## 採取したprofileをGo PGOへ戻す

GoはCPU pprof profileをPGOの入力として使えます。公式ドキュメントにも、代表的なworkloadから採ったprofileをbuildへ渡す流れがあります。

https://go.dev/doc/pgo

今回のbuildは次の形です。

```bash
go build \
  -pgo=/home/isucon/isutools-data/cpu_<capture-id>.pprof \
  -o isupipe .
```

どのprofileでbuildしたかをbuild infoへ残しました。profileは再現性に関わるbuild inputです。

`GOAMD64=v3`も試しましたが、441,453点でPGO v1に対する有意な改善を確認できず、採用しませんでした。

## アプリ以外に採用した変更

- nginxとGo間をsystemd管理のUnix domain socketへ変更
- MySQLのbufferと競技向けdurability設定を調整
- PowerDNSの過剰logを停止
- PowerDNS records検索indexを追加
- Echo production modeでcompact JSONを返す

`innodb_flush_log_at_trx_commit=2`などの設定は競技スコア優先です。停電時のtransaction耐久性が必要な本番へ、そのまま転用してはいけません。

## 採用しなかった変更

| 変更 | score・観測 | 判断 |
|---|---:|---|
| PowerDNS cache/thread増加 | score低下 | DNS攻撃のrampを含めて不採用 |
| 予約枠をmemoryだけで更新 | 417,657 | 不採用 |
| JSON gzip level 1 | 424,767 | 圧縮よりCPU負荷が勝ち不採用 |
| `GOAMD64=v3` | 441,453 | 有意な改善を確認できず不採用 |

gzipはresponseを約4.3分の1にできましたが、CPU消費が増えて公式scoreは下がりました。局所指標ではなく、`pass`、最終チェック、score、DNS attackerのrampで採否を決めています。

## `make bench`で保存とSCPまで行う

制御PC側のhost固有値は`isutools.mk`へ分離しました。

```make
REMOTE_HOST = user@example-host
WSL_DISTRO = isucon13
WINDOWS_USER = your-windows-user
RESULTS_DIR = $(HOME)/isutools-isucon13-results
LOCAL_PORT = 19197
```

```bash
make check
make bench
make tunnel
# http://127.0.0.1:19197/
```

`make bench`は次を行います。

1. HTTPS、MySQL、isutoolsのreadiness確認
2. resetとrun ID固定
3. 公式benchmark
4. result JSONの型検査
5. saveとprofile参照の解決
6. Windows共有directoryへstage
7. SCP
8. 最新snapshotの要約

serviceがactive、あるいは管理画面が200というだけでは成功扱いにしていません。公式benchの`pass`と最終チェックまでをgateにします。

## まとめ

12,136点から445,684点までの改善で役に立ったのは、次の流れでした。

1. 公式benchと計測artifactを同じrun IDで保存する
2. SQLとHTTPを回数・累計時間で並べる
3. N+1とDB往復を一括SQL・memory stateへ寄せる
4. ETag、compact JSON、Unix socketで大量の小さなコストを減らす
5. 代表的なCPU profileを行・call stack・flame graphで確認する
6. 同じprofileをPGOへ入力する
7. 局所指標が改善しても公式scoreが落ちれば戻す

isutoolsは自動チューニングツールではありません。SQL、HTTP、CPU、score、correctnessを同じrunへ揃え、「次に何を試すか」を決めるためのツールです。

詳細な環境構築、保存先、artifact、採用構成については本サイト版にも整理しています。

https://ekusiadadus.com/ja/blog/isucon13-445k-with-isutools

## 参考リンク

- [ISUCON13公式リポジトリ](https://github.com/isucon/isucon13)
- [wsl-isucon / isucon13](https://github.com/matsuu/wsl-isucon/tree/main/isucon13)
- [isutools](https://github.com/ekusiadadus/isutools)
- [Go Profile-guided optimization](https://go.dev/doc/pgo)
- [Go Diagnostics](https://go.dev/doc/diagnostics)
