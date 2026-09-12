---
title: "Astra単体とAstra＋GPT-5.6 SolをISUCON11予選で3時間比較した"
emoji: "⏱️"
type: "tech"
topics: ["isucon", "codex", "ai", "aws", "performance"]
published: false
---

Codexの**Astra単体**と、**AstraがGPT-5.6 Solの子をまとめる構成**を、各3時間・各1試行で比較しました。独立した競技サーバー3台＋ベンチ1台をそれぞれ用意し、計8 EC2で同時に実行しています。

結論は、**今回は単体が最終点とトークン効率で勝ち、複数は中盤の到達速度で先行した**、です。

この記事は要点版です。実験条件、全トークン内訳、子の稼働分析、公開用CSVは個人サイトにまとめました。

https://ekusiadadus.com/ja/blog/astra-vs-sol-multi-agent-isucon?utm_source=zenn&utm_medium=article&utm_campaign=astra_sol_isucon&utm_content=intro

## ISUCONを知らない方向けに

ISUCONは、渡されたWebサービスの機能を保ちながら、限られたサーバーで性能を改善する競技です。SQL、キャッシュ、CPU負荷、サーバー配置などに手を入れ、利用者を模したベンチマーカーで正しさと性能を測ります。

**今回測ったのはISUCON11予選の「ISUCONDITION」**です。椅子から届く状態データを保存し、履歴やグラフを表示するサービスで、継続的な書込みと読取り・集計を同時にさばきます。

ISUCON14の「ISURIDE」は、自動運転の椅子を呼ぶ配車サービスという[別の問題](https://github.com/isucon/isucon14/blob/main/docs/ISURIDE.md)です。今回は実行していません。

## 結果：最終点は単体が1.85倍

![各180分・各1試行。再起動後のスコアと、キャッシュ入力を含む総トークンの比較](/images/astra-sol-isucon/results-ja.png)

| 指標 | Astra単体 | Astra＋Sol |
|---|---:|---:|
| 再起動後スコア | **1,636,308** | 885,334 |
| 最終ベンチ判定 | pass | pass |
| 総トークン | **61,467,661** | 82,334,967 |
| 50万点への到達 | 2:31:05 | **2:04:05** |
| 子の累計 / 同時最大 | 0 / 0 | 24 / 3 |
| 通常の合格走行 | 50 | 46 |

複数側は50万点に約27分早く到達しましたが、終盤に単体が逆転しました。総トークンは複数が約34%多く、非キャッシュ入力では約2.20倍です。総トークンにはキャッシュされた入力が含まれ、請求金額の比率とは異なります。

## 子を呼べたことと、効率よく並列化できたことは別

複数側には、調査の分担、仮説単位の実装、専用worktree、実装同時2件までという運用を与えました。配備・初期化・ベンチ・ログ回収は、環境ごとの排他実行器を通しています。

native sessionの監査では、子24体すべてが指定した `gpt-5.6-sol` / highで、実装worktreeの分離と実装同時2件以内に反例は見つかりませんでした。

ただし、**子2体以上が同時に動いたのは3時間中約14分半**です。新しい文脈で起動しても、共通マニュアルの読み直しがあり、調査子の入力が大きくなる場面もありました。

次に改善したいのは子の上限より、親が測定している間に次の独立仮説を準備させる流れです。増員が通常の合格実験数を増やしたかまで追う必要がある、と感じました。

## この結果で言える範囲

各方式1回の改善試行なので、一般的なモデルの優劣は断定できません。EC2種別・台数はISUCON11の公式構成に合わせましたが、当時のAMIが取得不能だったため復元環境です。物理CPU型番と初期点にも差があります。

また、複数の選定版は同じcommitでも点数が変動し、再起動後には9件の通信エラーと71件のtimeoutが残りました。両方passですが、さらに再起動してデータを読戻す検査とブラウザの追試は未実施です。

実験後は全8 EC2と関連EBS・専用SG・キーペアをdestroyし、削除を確認しました。

## 時間推移と詳しい分析

詳細版では、**何分で逆転したか、親だけのトークンを見てはいけない理由、子の実稼働、最後に選ばれた解法の違い**を図と表で整理しています。集計JSONと全候補のCSVも添付しました。

[個人サイトで詳しい実験記録を読む](https://ekusiadadus.com/ja/blog/astra-vs-sol-multi-agent-isucon?utm_source=zenn&utm_medium=article&utm_campaign=astra_sol_isucon&utm_content=outro)

[English version](https://ekusiadadus.com/en/blog/astra-vs-sol-multi-agent-isucon?utm_source=zenn&utm_medium=article&utm_campaign=astra_sol_isucon&utm_content=english)
