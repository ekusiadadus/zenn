---
title: "ISUCON 1〜14の技術スタックを全部調べて、isutoolsを過去問対応した"
emoji: "🧭"
type: "tech"
topics: ["isucon", "go", "mysql", "nginx", "performance"]
published: false
---

ISUCONの過去問で計測ツールを使おうとすると、回ごとにWebフレームワーク、DB、リバースプロキシが違います。

「だいたいGo + MySQL + nginx」として扱うだけでは、ISUCON5本選のPostgreSQL、ISUCON8予選のH2O、ISUCON10本選のEnvoy、ISUCON12予選のtenant別SQLiteなどを取りこぼします。そこで、公式に公開されているISUCON 1〜14の問題リポジトリをすべて調べ、Go実装、DB / storage、HTTP frontend / proxy、外部サービスを棚卸ししました。

調査記録とファイル単位の根拠は、次のIssueに残しています。

[Support every published ISUCON Go stack, database, and proxy — isutools #45](https://github.com/ekusiadadus/isutools/issues/45)

この記事では、その長い調査結果から「過去問を横断すると何が見えるか」と「計測ツール側をどう設計したか」をまとめます。最後に、AWS、WSL2、Vagrant、Dockerで過去問を動かすためのリンクも載せます。

## 先に結論

調べて分かったのは、次の4点です。

1. **Go実装がない回もある**: ISUCON1と2は、公開リポジトリにGoの参考実装がありません。Go middlewareだけで全回対応とは言えません。
2. **DBはMySQLだけではない**: PostgreSQL、SQLite、Redisも、初期実装の主要data pathに登場します。
3. **frontendはnginxだけではない**: Apache httpd、H2O、Envoy、nginx streamのL4 proxyまであります。
4. **「ログを読めた」と「対応済み」は違う**: format、時間単位、fixture、設定検証、security boundaryを製品ごとに明示する必要があります。

この調査をもとに、[isutools v1.6.0](https://github.com/ekusiadadus/isutools/releases/tag/v1.6.0)では、公開済みISUCONのGo router、DB / storage、proxy logを扱うためのcontractを追加しました。ただし、全過去問VMでの実機認証を主張するものではありません。この境界についても後半で説明します。

## ISUCON 1〜14の技術スタック一覧

調査日は2026年8月14日です。問題アプリ、ベンチマーカー、ポータル、運営側proxyを区別し、packageがインストールされているだけの場合は「初期Go実装が利用している」と数えていません。

| 回 | Go実装 / router | DB / storage | frontend / 主な外部要素 |
|---|---|---|---|
| [ISUCON1](https://github.com/isucon/isucon) | なし。Perl / Ruby / Node.js | MySQL | 統一frontendは公開repoで未確認、supervisord |
| [ISUCON2](https://github.com/isucon/isucon2) | なし | MySQL | 統一proxyは未確認、PHP sampleはApache httpd :5000 |
| [ISUCON3予選](https://github.com/isucon/isucon3/tree/master/qualifier) | Gorilla mux | MySQL、memcached session | Go app :5000、画像をlocal filesystemへ保存 |
| [ISUCON3本選](https://github.com/isucon/isucon3/tree/master/final) | Gorilla mux | MySQL | Apache httpd :80 → app :5000 |
| [ISUCON4予選](https://github.com/isucon/isucon4/tree/master/qualifier) | Martini | MySQL 5.5 | nginx → app :8080 |
| [ISUCON4本選](https://github.com/isucon/isucon4/tree/master/final) | Martini | Redis | asset、広告、counter、local click log |
| [ISUCON5予選](https://github.com/isucon/isucon5-qualify) | Gorilla mux | MySQL 5.6 | nginx、event / image role |
| [ISUCON5本選](https://github.com/isucon/isucon5-final) | Gorilla mux | PostgreSQL 9.4 | nginx、外部weather API、API / image role |
| [ISUCON6予選](https://github.com/isucon/isucon6-qualify) | Gorilla mux | MySQL 5.7 | nginx、isuda / isutar / isupam |
| [ISUCON6本選](https://github.com/isucon/isucon6-final) | Goji v2 / pat | MySQL | nginx HTTP + stream L4、TLS、SSE、Consul、5 nodes |
| [ISUCON7予選](https://github.com/isucon/isucon7-qualify) | Echo pre-v4 | MySQL | nginx、chat application |
| [ISUCON7本選](https://github.com/isucon/isucon7-final) | Gorilla mux + WebSocket | MySQL | nginx → app :5000、realtime game |
| [ISUCON8予選](https://github.com/isucon/isucon8-qualify) | Echo pre-v4 | MySQL | H2O → app :8080、PHP FastCGI variant |
| [ISUCON8本選](https://github.com/isucon/isucon8-final) | httprouter | MySQL 8 | nginx、bank / logger blackbox、4 servers |
| [ISUCON9予選](https://github.com/isucon/isucon9-qualify) | chi v5 | MySQL 8 | nginx / TLS、payment / shipment service、最大3 nodes |
| [ISUCON9本選](https://github.com/isucon/isucon9-final) | Goji v2 / pat | MySQL 8 | nginx、gRPC + grpc-gatewayのpayment blackbox |
| [ISUCON10予選](https://github.com/isucon/isucon10-qualify) | Echo pre-v4 | MySQL | nginx、frontendとAPI serverを分離 |
| [ISUCON10本選](https://github.com/isucon/isucon10-final) | Echo v4 | MySQL 8 | Envoy、protobuf / service API、Web Push |
| [ISUCON11予選](https://github.com/isucon/isucon11-qualify) | Echo v4 | MariaDB 10.3 | nginx、JIA external API、image upload |
| [ISUCON11本選](https://github.com/isucon/isucon11-final) | Echo v4 | MySQL | nginx、course / submission application |
| [ISUCON12予選](https://github.com/isucon/isucon12-qualify) | Echo v4 | MySQL 8 + tenant別SQLite | nginx、JWT / blackauth |
| [ISUCON12本選](https://github.com/isucon/isucon12-final) | Echo v4 | MySQL 8 | nginx、game / present workload |
| [ISUCON13](https://github.com/isucon/isucon13) | Echo v4 | MySQL 8 + PowerDNS用MySQL | nginx / TLS、PowerDNS、DNS water-torture、3 servers |
| [ISUCON14](https://github.com/isucon/isucon14) | chi v5 | MySQL 8 | nginx、payment mock、optional SSE、3 servers |

初期の回で統一frontend設定が見つからなかった場合は、「proxyなし」ではなく「公開リポジトリでは未確認」としました。公開物にない当日の構成を推測で補わないためです。

## Goのrouterは時代ごとに変わっている

Go参考実装のrouterだけを並べても、Gorilla mux、Martini、Goji v2 / pat、Echo pre-v4、Echo v4、httprouter、chi v5と変遷しています。

計測middlewareで重要なのは、raw URLではなく、登録したroute templateを取得することです。例えば`/users/123`と`/users/456`を別endpointとして集計すると、IDの数だけcardinalityが増えます。URLにtokenが含まれていれば、情報漏えいにもつながります。

そのため、isutoolsではrouterごとにroute identityの取り方を固定しました。

| router | route identity |
|---|---|
| `net/http` | Go 1.22の`Request.Pattern` |
| Gorilla mux | routing後の`GetPathTemplate` |
| Martini / Goji v2 / httprouter | route登録時に渡した定数 |
| Echo v3 / v4 / v5 | `Context.Path` |
| chi v5 | routing後の`RoutePattern` |

どのadapterもraw URL pathへfallbackしません。計測不能な場合は、もっともらしい値を捏造するより、欠損として扱う方を選びます。

ISUCON1と2にはGo参考実装がないため、Go middlewareは挿せません。この2回でも、proxy access logとDB collectorは独立して使える、という形に分けました。

## MySQL以外のdata storeをSQLに偽装しない

MySQL / MariaDB、PostgreSQL、SQLiteは`database/sql`のquery timing、count、errorを同じcontractで計測できます。ただし、MySQLで取得できるschema情報や`EXPLAIN`の意味を、PostgreSQLやSQLiteにもそのまま当てはめてはいけません。

特にISUCON12予選は、system DBがMySQLで、tenantごとのDBがSQLite fileです。接続先が1個とは限らず、file数やpathを無制限にlabelへ入れない設計が必要になります。

ISUCON4本選のRedisは、SQLではありません。そこで、Redisは独立したcollectorとして次だけを保持します。

- command名
- count
- total / average / p95
- error count

key、value、引数、接続文字列、error本文は保存しません。SQL collectorへ無理に寄せないことで、metricの意味と秘密情報の境界を保ちます。

## proxy logは製品名と時間単位を明示する

最初の実装では、JSONかLTSVかを行頭で判定していました。しかし、同じJSONでもdurationの単位は製品ごとに異なります。

- Caddy: 秒
- Traefik: ナノ秒
- IIS W3Cの`time-taken`: ミリ秒
- 独自JSON: field名で`_ns` / `_us` / `_ms` / `_sec`を明示

ここを推測すると、p95や累計時間が1000倍、10億倍単位で壊れます。そのため、新規設定では`ISUTOOLS_ACCESS_LOG_FORMAT`を明示し、decoder boundaryでnanosecondsへ変換するようにしました。

また、ISUCON6本選のnginx stream logはTCP connectionの記録です。HTTP method、URI、statusが存在しないため、HTTP access eventへ変換せず、L4 connection eventとして分離します。

## User Flowをproxy headerだけに依存させない

複数endpointをたどるUser Flowをaccess logから作る場合、sessionやscenarioのlabelが必要になります。しかし、public requestの`X-Isutools-Session`をそのまま信頼すると、clientが任意のlabelを注入できます。CookieやAuthorizationをlogへ書くのも危険です。

現在はapplication middlewareをprimary sourceにしています。

1. 対象CookieをHMACで疑似ID化する
2. raw Cookieやsession tokenは保存しない
3. routerが確定したboundedなroute templateだけを記録する
4. public client由来のflow headerは信用しない
5. middlewareとproxyの観測を同時集計しない

`ISUTOOLS_FLOW_SOURCE=auto`はmiddleware観測を優先し、なければ従来のproxy response labelへfallbackします。`middleware`、`proxy`、`off`も明示できます。

proxy logとmiddleware eventを足し合わせないのは、同じrequestを二重計上しないためです。

## 「対応済み」を3段階に分けた

全製品を一つのチェックマークで「対応」と書くと、検証の強さが分かりません。そこで、少なくとも次を分けています。

| 表記 | 意味 |
|---|---|
| native-config-validated | versionを固定した実binaryで設定検証を実施 |
| schema-compatible | 公式formatに基づくfixtureとdecoder contractを検証 |
| L4-limited | TCP connection metricだけを扱い、HTTP fieldは持たない |

v1.6.0では、nginx 1.28.0、Apache httpd 2.4.65、H2O 2.2.5、Envoy 1.34.13 / 1.33.11、Caddy 2.10.2でnative config validationを行いました。HAProxy、Traefik、lighttpd、Varnish、Apache Traffic Server、IIS、Squid、OpenLiteSpeedなどは、製品別fixtureまたはformat contractによるschema-compatibleです。

これは、過去の全VM imageで検証済みという意味ではありません。例えばPostgreSQLの実driver integrationが通っていても、それだけでISUCON5本選の当時環境をcertifiedとは呼びません。

「その製品のlogを正しい単位で解釈できる」と「その大会環境でbenchmarkまで通した」を分けて書くようにしました。

## 過去問環境を作るためのリンク

ソースを読むだけでなく、実際に過去問を動かす場合は次が入口になります。対応回やimageは更新・削除される可能性があるため、利用時点のREADMEも確認してください。

| 環境 | 向いている用途 | リンク |
|---|---|---|
| 公式まとめ | AWS、WSL2、Vagrant、Dockerなどの選択肢を比較する | [ISUCON公式Blog: 過去問環境の構築](https://isucon.net/archives/54946542.html) |
| 公式ソース | 各回のmanual、参考実装、benchmarker、provisioningを確認する | [GitHub: isucon organization](https://github.com/isucon) |
| AWS | 本番に近いLinux VMを短時間で用意する。ISUCON5予選〜14のAMI / Packer | [matsuu/aws-isucon](https://github.com/matsuu/aws-isucon) |
| WSL2 | Windows PC上でISUCON9予選〜14を動かす | [matsuu/wsl-isucon](https://github.com/matsuu/wsl-isucon) |
| Vagrant | 1台構成を手元で再現する。ISUCON2〜14の一部 | [matsuu/vagrant-isucon](https://github.com/matsuu/vagrant-isucon) |
| ISUCON14公式 | AMI、Packer / Ansible、Go / Perl向けDocker Compose | [isucon/isucon14](https://github.com/isucon/isucon14) |
| 練習用アプリ | 公式大会の過去問とは別に、Docker Composeで基本的な性能改善を練習する | [catatsuy/private-isu](https://github.com/catatsuy/private-isu) |

AWSは利用料金が発生します。古いAMIは予告なく利用できなくなる可能性があり、公式repoに含まれるTLS証明書も期限切れの場合があります。

競技当時と同じCPU、台数、network、外部serviceまで再現できるとは限りません。ローカルDockerで動いた結果と、本番相当VMでのscoreは分けて扱う必要があります。

自分がISUCON13をWSL2で試したときは、次の環境を使いました。

- [matsuu/wsl-isucon / isucon13](https://github.com/matsuu/wsl-isucon/tree/main/isucon13)
- [isutoolsのISUCON13 WSL2導入例](https://github.com/ekusiadadus/isutools/tree/main/examples/isucon13-wsl)

ISUCON14には、公式repoのprovisioningを使う手順と、isutools側の最小導入例があります。

- [ISUCON14公式リポジトリ](https://github.com/isucon/isucon14)
- [isutoolsのISUCON14導入例](https://github.com/ekusiadadus/isutools/tree/main/examples/isucon14)

## まとめ

ISUCONの歴代構成は、単純な「Go + MySQL + nginx」ではありませんでした。

- Go routerはGorilla mux、Martini、Goji、Echo、httprouter、chiへ変化した
- MySQL系以外にPostgreSQL、SQLite、Redisが主要data pathとして登場した
- frontendにはApache、H2O、Envoy、nginx stream L4があった
- 外部service、WebSocket、SSE、DNS、複数node構成も計測境界へ影響した

この調査で一番大きかった学びは、対応製品の数よりも、**何を、どの単位で、どこまで検証したかを明示すること**でした。

過去問で使う場合も、まず公式benchmarkの`pass`とscoreを保存し、そのrunにSQL、HTTP、proxy log、runtime profileを結び付けます。計測できたことと、性能が改善したことは別です。最終的な採否は、同じ条件のbenchmarkとcorrectnessで決めます。

## 参考リンク

- [調査Issue #45](https://github.com/ekusiadadus/isutools/issues/45)
- [isutools v1.6.0 release](https://github.com/ekusiadadus/isutools/releases/tag/v1.6.0)
- [ISUCON compatibility matrix](https://github.com/ekusiadadus/isutools/blob/main/docs/isucon-compatibility.md)
- [isutools](https://github.com/ekusiadadus/isutools)
