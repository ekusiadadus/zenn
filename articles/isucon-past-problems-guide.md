---
title: "ISUCON 1〜14の過去問環境と技術スタックまとめ"
emoji: "🗂️"
type: "tech"
topics: ["isucon", "go", "mysql", "nginx", "performance"]
published: true
---

ISUCONの過去問をやってみたいと思っても、回ごとにリポジトリや環境構築方法が分かれていて、どれを選べばよいか迷います。

そこで、公式に公開されているISUCON 1〜14の問題を、Go実装、DB、Webサーバー・プロキシの3点で一覧にしました。AWS、WSL2、Vagrant、Dockerで試すためのリンクもまとめています。

## まずは環境を選ぶ

すぐに過去問を始めたい場合は、手元の環境に合わせて次から選ぶのが分かりやすいです。

| 環境 | 収録されている主な問題 | リンク |
|---|---|---|
| AWS | ISUCON5〜14の一部（13・14を含む） | [matsuu/aws-isucon](https://github.com/matsuu/aws-isucon) |
| WSL2 | ISUCON9〜14の一部（13・14を含む） | [matsuu/wsl-isucon](https://github.com/matsuu/wsl-isucon) |
| Vagrant | ISUCON2〜14の一部 | [matsuu/vagrant-isucon](https://github.com/matsuu/vagrant-isucon) |
| ISUCON14公式 | AMI、Packer / Ansible、Docker Compose | [isucon/isucon14](https://github.com/isucon/isucon14) |
| Dockerで練習 | 公式大会とは別の定番練習問題 | [catatsuy/private-isu](https://github.com/catatsuy/private-isu) |

環境ごとの選択肢は、[ISUCON公式Blogの過去問環境まとめ](https://isucon.net/archives/54946542.html)にも整理されています。

迷った場合は、Windowsなら`wsl-isucon`、AWSを使えるなら`aws-isucon`、まず1台でWeb性能改善を練習したいなら`private-isu`が始めやすいと思います。

## ISUCON 1〜14の技術スタック一覧

調査日は2026年8月14日です。各リンクから公式の問題リポジトリを開けます。

| 回 | Go実装 | DB / storage | Webサーバー・プロキシ |
|---|---|---|---|
| [ISUCON1](https://github.com/isucon/isucon) | なし | MySQL | 公開repoでは統一設定を確認できず |
| [ISUCON2](https://github.com/isucon/isucon2) | なし | MySQL | PHP sampleはApache httpd |
| [ISUCON3予選](https://github.com/isucon/isucon3/tree/master/qualifier) | Gorilla mux | MySQL、memcached | 公開repoでは統一設定を確認できず |
| [ISUCON3本選](https://github.com/isucon/isucon3/tree/master/final) | Gorilla mux | MySQL | Apache httpd |
| [ISUCON4予選](https://github.com/isucon/isucon4/tree/master/qualifier) | Martini | MySQL 5.5 | nginx |
| [ISUCON4本選](https://github.com/isucon/isucon4/tree/master/final) | Martini | Redis | appが直接配信 |
| [ISUCON5予選](https://github.com/isucon/isucon5-qualify) | Gorilla mux | MySQL 5.6 | nginx |
| [ISUCON5本選](https://github.com/isucon/isucon5-final) | Gorilla mux | PostgreSQL 9.4 | nginx |
| [ISUCON6予選](https://github.com/isucon/isucon6-qualify) | Gorilla mux | MySQL 5.7 | nginx |
| [ISUCON6本選](https://github.com/isucon/isucon6-final) | Goji v2 / pat | MySQL | nginx HTTP + stream L4 |
| [ISUCON7予選](https://github.com/isucon/isucon7-qualify) | Echo pre-v4 | MySQL | nginx |
| [ISUCON7本選](https://github.com/isucon/isucon7-final) | Gorilla mux + WebSocket | MySQL | nginx |
| [ISUCON8予選](https://github.com/isucon/isucon8-qualify) | Echo pre-v4 | MySQL | H2O |
| [ISUCON8本選](https://github.com/isucon/isucon8-final) | httprouter | MySQL 8 | nginx |
| [ISUCON9予選](https://github.com/isucon/isucon9-qualify) | chi v5 | MySQL 8 | nginx / TLS |
| [ISUCON9本選](https://github.com/isucon/isucon9-final) | Goji v2 / pat | MySQL 8 | nginx |
| [ISUCON10予選](https://github.com/isucon/isucon10-qualify) | Echo pre-v4 | MySQL | nginx |
| [ISUCON10本選](https://github.com/isucon/isucon10-final) | Echo v4 | MySQL 8 | Envoy |
| [ISUCON11予選](https://github.com/isucon/isucon11-qualify) | Echo v4 | MariaDB 10.3 | nginx |
| [ISUCON11本選](https://github.com/isucon/isucon11-final) | Echo v4 | MySQL | nginx |
| [ISUCON12予選](https://github.com/isucon/isucon12-qualify) | Echo v4 | MySQL 8 + tenant別SQLite | nginx |
| [ISUCON12本選](https://github.com/isucon/isucon12-final) | Echo v4 | MySQL 8 | nginx |
| [ISUCON13](https://github.com/isucon/isucon13) | Echo v4 | MySQL 8 + PowerDNS用MySQL | nginx / TLS |
| [ISUCON14](https://github.com/isucon/isucon14) | chi v5 | MySQL 8 | nginx |

ISUCON1と2には、公開リポジトリ上のGo参考実装がありません。初期の回でWebサーバーの統一設定が見つからなかったものは、「使っていない」と断定せず「公開repoでは未確認」としています。

## 目的別に選ぶなら

### 最近の構成を触りたい

- [ISUCON13](https://github.com/isucon/isucon13): Echo v4、MySQL、nginx、PowerDNS、3台構成
- [ISUCON14](https://github.com/isucon/isucon14): chi v5、MySQL、nginx、決済mock、3台構成

どちらも公式repoにbenchmarkerやprovisioningが含まれています。ISUCON14はDocker Composeでも起動できます。

### MySQL以外も触りたい

- [ISUCON4本選](https://github.com/isucon/isucon4/tree/master/final): Redisが中心
- [ISUCON5本選](https://github.com/isucon/isucon5-final): PostgreSQL
- [ISUCON12予選](https://github.com/isucon/isucon12-qualify): MySQL + tenant別SQLite

### nginx以外も見たい

- [ISUCON3本選](https://github.com/isucon/isucon3/tree/master/final): Apache httpd
- [ISUCON8予選](https://github.com/isucon/isucon8-qualify): H2O
- [ISUCON10本選](https://github.com/isucon/isucon10-final): Envoy
- [ISUCON6本選](https://github.com/isucon/isucon6-final): nginx streamを使ったL4 proxy

### WebSocketや複数台構成を触りたい

- [ISUCON6本選](https://github.com/isucon/isucon6-final): 5台構成、SSE、Consul
- [ISUCON7本選](https://github.com/isucon/isucon7-final): WebSocketを使うリアルタイムゲーム
- [ISUCON13](https://github.com/isucon/isucon13): DNS負荷を含む3台構成
- [ISUCON14](https://github.com/isucon/isucon14): 決済mockとoptional SSEを含む3台構成

## 実際にISUCON13を解いてみた例

自分は[matsuu/wsl-isuconのISUCON13環境](https://github.com/matsuu/wsl-isucon/tree/main/isucon13)を使い、公式ベンチを回しながらボトルネックを調べました。

![ISUCON13の公式ベンチ後に保存した計測レポート](https://raw.githubusercontent.com/ekusiadadus/isutools/main/docs/images/isutools-isucon13-specialist-20260815.png)

SQLやHTTPの実行回数、累計時間、p95、DB pool、CPUなどを、公式ベンチのscoreと一緒に保存しています。計測には自作の[isutools](https://github.com/ekusiadadus/isutools)を使いました。

環境への導入例は[ISUCON13 WSL2 example](https://github.com/ekusiadadus/isutools/tree/main/examples/isucon13-wsl)、改善内容は[ISUCON13を12,136点から445,684点まで改善した記録](https://ekusiadadus.com/ja/blog/isucon13-445k-with-isutools)にまとめています。

## 環境構築時の注意

- AWSのAMIを使う場合は料金が発生する
- 古いAMIは利用できなくなる可能性がある
- 公式repo内のTLS証明書は期限切れの場合がある
- Dockerや1台構成は、競技当時の台数・network・外部serviceと同じとは限らない
- 環境が起動しただけでなく、公式benchmarkerの`pass`まで確認する

ローカルDockerのscoreと、本番相当の複数VMでのscoreは分けて考えた方が安全です。

## まとめ

- Windowsなら`wsl-isucon`、AWSなら`aws-isucon`から始めやすい
- 最近の構成ならISUCON13・14、手軽な練習なら`private-isu`が候補
- 過去問ごとにGo framework、DB、Webサーバーが異なるので、練習したい技術から選ぶのも面白い

最後に、入口になるリンクをもう一度まとめます。

- [ISUCON公式Blog: 過去問環境の構築](https://isucon.net/archives/54946542.html)
- [GitHub: isucon organization](https://github.com/isucon)
- [matsuu/aws-isucon](https://github.com/matsuu/aws-isucon)
- [matsuu/wsl-isucon](https://github.com/matsuu/wsl-isucon)
- [matsuu/vagrant-isucon](https://github.com/matsuu/vagrant-isucon)
- [catatsuy/private-isu](https://github.com/catatsuy/private-isu)

技術スタックの調査根拠は[GitHub Issue #45](https://github.com/ekusiadadus/isutools/issues/45)に残しています。


