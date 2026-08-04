---
title: "1行で組み込めるISUCON用オールインワンプロファイラ isutools を作った"
emoji: "⚡"
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["isucon", "go", "mysql", "nginx", "パフォーマンス"]
published: false # 公開設定（falseにすると下書き）
---

ISUCON の計測は道具が分散しがちです。SQL は pt-query-digest、アクセスログは alp、CPU は top と pprof — ベンチのたびに端末を行き来して、結果はどこにも残らない。

これを 1つのダッシュボードに集約する Go 製 OSS を作りました。

https://github.com/ekusiadadus/isutools

- 組み込みは実質 **1行**(`sqlx.Open` の書き換えのみ)
- SQL / HTTP / nginx ログ / プロセス CPU / DB スキーマ / pprof を1画面に
- ベンチごとに**スコアと git リビジョン付きスナップショット**が自動で残り、実行間 **diff** も取れる
- 計測オーバーヘッドは ABBA 計測で **-0.58%(誤差内)**
- MIT ライセンス

![isutools ダッシュボードの実行履歴](https://ekusiadadus.com/images/private-isu/isutools-runs.png)

## 組み込み

```go
// before
db, err := sqlx.Open("mysql", dsn)

// after — SQL計測 + localhost:19191 に管理サーバが立つ
db, err := sqlx.Open(isutools.SQLDriverName("mysql"), dsn)
```

運用はベンチスクリプトに2つ足すだけです。

```bash
curl -XPOST localhost:19191/reset
./run-benchmark
curl -XPOST "localhost:19191/save?score=123456"
```

## 「未設定の定石」を自動検出する advisor

MySQL・nginx・OS・Go の設定を読んで、ISUCON の定石なのに未設定のものを指摘します。private-isu では初回実行で「interpolateParams なし」「gzip なし」「buffer_pool がデータ量の 1/9」を即指摘 → 3件適用で +16% でした。

![advisor セクション](https://ekusiadadus.com/images/private-isu/isutools-advisor.png)

## 実戦: private-isu を1日で 0 → 541,650

このツールで計測しながら private-isu(Go 実装)を1日チューニングした記録です。

| ステップ | 施策 | スコア |
|---|---|---:|
| 初期状態 | Go 実装のまま | 0 |
| ① | インデックス + N+1 一括化 | 19,290 |
| ④ | write-through 配置バグ修正 | 111,756 |
| ⑤ | 接続チューニング | 299,668 |
| ⑧ | コメントキャッシュ | 412,057 |
| ⑨ | インデックス追加でオプティマイザ退行 | 140,914 |
| ⑩ | 事前レンダリング + 退行修正 | **541,650** |

「インデックスを1本追加したらタイムラインの JOIN が **260倍遅くなった**」事故を diff ビューで数分で特定できたのが個人的ハイライトです。

**完全版(全10ステップの数字と失敗談)は本サイトに書きました:**

- 導入と全機能: https://ekusiadadus.com/ja/blog/isutools-isucon-profiler
- 50万点までの全記録: https://ekusiadadus.com/ja/blog/private-isu-500k-with-isutools

バグ報告・機能要望は [GitHub Issues](https://github.com/ekusiadadus/isutools/issues) へ。Star をもらえると開発の励みになります。
