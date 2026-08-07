---
title: "ISUCONで「速くなった理由」を残すための1修正1ベンチ運用"
emoji: "📊"
type: "tech"
topics: ["isucon", "go", "mysql", "performance", "observability"]
published: false
---

ISUCONでスコアが上がっても、変更が効いた理由まで説明できるとは限りません。ベンチマークには揺れがあり、スコアが高いrunでも仕様不整合を含む場合があります。また、平均1msのSQLでも10万回呼ばれれば、1回だけ遅いSQLより大きな負荷になります。

この記事では、ISUCON14のGo実装を改善した際に使った、次の運用を整理します。

1. 変更を1つの意図に限定します。
2. 同じ計測境界でベンチマークを1回実行します。
3. スコアとcorrectnessを分けて判定します。
4. SQL、HTTP、DB connection pool、CPUなどを同じartifactへ保存します。
5. 採用またはrevertの理由を残します。

ISUCON14固有のマッチング実装を網羅する記事ではありません。別のISUCON問題や通常のWebアプリケーションでも再利用しやすい、計測と判断の方法に絞ります。

## スコアだけでは採否を決めない

今回のベンチでは、次の3つを別々に扱いました。

- **score**: ベンチマーカーが算出した性能指標
- **correctness**: 仕様違反や不整合を示すerror code
- **measurement health**: 計測の欠損、revision、dirty状態、計測区間

たとえば、ある変更では637,737点まで伸びましたが、付近の椅子一覧に関するerror code 30が13件発生しました。raw scoreは高くても、アプリケーションの観測整合性を悪化させているため採用していません。

一方、567,276点のrunは同じerror codeが0件でした。性能改善では最高値だけを残すのではなく、仕様を満たしたrollback地点を残すことが重要でした。

## 計測区間を固定する

計測には、Goアプリケーションへ組み込んだ[isutools](https://github.com/ekusiadadus/isutools)を使いました。SQL、HTTP、nginx、DB connection pool、CPU、ホスト資源などを、1つのベンチマーク区間として保存できます。

ベンチスクリプトから行う操作は、次の3段階です。

```bash
curl -fsS -X POST http://127.0.0.1:19191/reset
./run-benchmark
curl -fsS -X POST \
  'http://127.0.0.1:19191/save?score=569078&pass=true'
```

重要なのはコマンドの短さではなく、`reset`と`save`の間だけを比較対象にすることです。initialize前の処理や、ベンチ終了後のログ回収がrunごとに混ざると、差分の意味が変わります。

保存時には次の情報も確認します。

- アプリケーションのcommit
- artifactに埋め込まれたrevisionとdirty状態
- collectorの欠損やdegraded表示
- ベンチマーカーのpass/failとerror map

今回の569,078点artifactはアクセスログのkey上限により`partial=true`で、埋め込みrevisionもdirtyでした。そのため、artifactだけで完全なprovenanceを主張せず、実機のアプリケーションcommit `7fc4a4c`、ベンチログ、artifact IDを対応付けています。計測ツールが警告を出したときは、警告を消すより証明範囲を狭める方が安全です。

## レポートは「遅い順」だけで読まない

改善候補は次の順で見ました。

### 1. correctnessと計測の健全性

failしたrunや計測区間が壊れたrunは、性能比較の採用候補から外します。ただし、原因を追跡できるようartifactとcommitは残します。

### 2. 累計需要

SQLとHTTPは、平均時間だけでなく`count × latency`で生じる累計時間を見ます。短い処理でも呼び出し回数が多ければ、DB poolやCPUを長時間使います。

### 3. 待ちと実行を分ける

HTTPの累計時間が大きくても、long pollingの待機時間ならCPU負荷とは限りません。同様に、DB pool waitが大きいことは、接続数を増やすべきという意味ではありません。

### 4. 資源の余力

CPU、disk utilization、I/O PSI、DBの同時実行数を確認します。CPUに余裕があるのにDB poolだけが詰まっている場合は、SQL回数やtransactionの保持時間を先に疑います。

## 実例: 履歴APIのN+1をcacheで除去した

567,276点のartifactでは、`GET /api/app/rides`が完了済みrideを返すたびに、chairとownerを個別に取得していました。

```sql
SELECT * FROM chairs WHERE id = ?;
SELECT * FROM owners WHERE id = ?;
```

これらは1回あたりは短いSQLです。しかし、1回のベンチでそれぞれ90,348回呼ばれ、2本の合計で約155.7秒を使っていました。

アプリケーションにはすでにchairのID cacheがあったため、履歴APIからもそれを再利用しました。ownerについてはID cacheを追加し、initialize時に他のcacheと同時に消去します。履歴レスポンスで使うchair名、model、owner名はベンチ中に更新されないため、更新経路のない値だけを対象にしました。

変更前後の同じ項目は次のとおりです。

| 指標 | 変更前 | 変更後 |
|---|---:|---:|
| score | 567,276 | **569,078** |
| `chairs WHERE id` | 90,348回 / 82.14秒 | **11回 / 0.002秒** |
| `owners WHERE id` | 90,348回 / 73.56秒 | **3回 / 0.001秒** |
| `GET /api/app/rides` 平均 | 6.62ms | **4.06ms** |
| 同p95 | 33.55ms | **16.78ms** |
| DB pool wait回数 | 65,991 | **50,323** |
| DB pool wait合計 | 922.15秒 | **575.28秒** |

スコア差は+1,802点、約+0.32%です。1回の比較だけで、この差を厳密な因果効果とは扱えません。一方で、対象SQLが約18万回から14回へ減り、履歴APIのp95とpool waitも同じ方向へ改善しています。実装の不変条件をテストし、correctness gateも通過したため、この変更は採用しました。

この例から分かるのは、「スコア差が小さいので無意味」でも「SQLが減ったので大幅な高速化」でもありません。局所的なN+1は除去できましたが、サービス全体の次の制約は別にある、という判断です。

## DB pool waitを見て接続数を増やした失敗

DB poolが100接続すべて使われていたため、200へ増やす実験も行いました。しかし、スコアは567,276点から546,855点へ下がりました。

接続数を増やすと、待っていた処理がすべて有用な処理として進むとは限りません。今回のアプリケーションはnotification pollingが多く、同時に通すrequestを増やすとMySQL側の競合も増えます。この変更はrevertし、SQL回数とtransaction保持時間を減らす方へ戻しました。

pool waitは原因そのものではなく、待ちが発生している場所です。次の点を併記しないと判断を誤ります。

- pool slotを使っているendpoint
- connectionを保持している時間
- MySQLとアプリケーションのCPU
- 完了した有用仕事の数
- scoreとcorrectness

## 1修正1ベンチを崩さない

変更をまとめると、スコアが変わっても理由を分解できません。今回の履歴は、次の単位で残しました。

```text
仮説
  ↓
1つの意図を持つcommit
  ↓
reset → benchmark → save
  ↓
score / correctness / artifactを確認
  ↓
採用 または revert
```

設定変更も同じです。Goコード、MySQL設定、nginx設定を同時に変えず、できる限り別runにしました。ベンチの揺れが疑わしい場合は同じrevisionを再実行し、1回の最高値ではなく分布を確認します。

## 計測ツールが自動では判断できないこと

isutoolsから直接確認できるのは、SQL回数、HTTP latency、DB pool wait、CPU、I/O、変更前後の差分などです。一方で、次のような内容はアプリケーションの仕様を読まないと判断できません。

- cacheしてよい値とinvalidate条件
- clientが状態変化を観測する順序
- scoreに寄与する業務上のcritical path
- ベンチマーカー固有のerror codeの意味
- 一時的な不整合が許容される時間

計測結果は修正候補を絞るための根拠です。アプリケーション固有の正しさは、コード、仕様、ベンチマーカーログと突き合わせます。

## まとめ

Webアプリケーションの性能改善では、最も遅い1リクエストだけでなく、短い処理の回数、poolでの待ち、資源の余力、correctnessを同じrunで確認する必要があります。

今回特に役立った原則は次の4つです。

- scoreより先にcorrectnessと計測の健全性を確認します。
- averageだけでなく、countと累計需要を見ます。
- pool waitを接続数増加の指示とは解釈しません。
- 失敗runとrevert理由もartifactとして残します。

ISUCON14で行ったマッチング、通知、coordinate、payment、cacheの全変更と、95,656点から567,276点までの時系列、割り当て比較GIFは[本サイトのケーススタディ](https://ekusiadadus.com/ja/blog/isucon14-542k-with-isutools)にまとめています。導入方法とソースコードは[isutoolsのGitHubリポジトリ](https://github.com/ekusiadadus/isutools)で確認できます。
