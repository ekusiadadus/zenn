---
title: "MySQL WorkBench でスロークエリをとって ISUCON 頑張る" # 記事のタイトル
emoji: "🥥" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["ISUCON", "Golang"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# MySQL WorkBench でスロークエリをとって ISUCON 頑張る

MySQL WorkBench を 仕事で最近使うようになりました。
MWB を触っているうちに、結構いろいろなことができることに気が付きました。
例えば、ER 図を作成したり、スロークエリログをとることです。

また、[達人が教える Web パフォーマンスチューニング](https://gihyo.jp/book/2022/978-4-297-12846-3)が発売されて、Twitter で話題になっていて気になっていました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">「達人が教えるWebパフォーマンスチューニング ～ISUCONから学ぶ高速化の実践」について読み始めて、色々突っ込みどころというか、間違いが満載なので、Qiitaにまとめでも書こうかと下書きで書いているのですが、あまりに多すぎて、自分の時間の優先順位を思い出して書くのを止めようかと…</p>&mdash; Yoichiro Takehora (竹洞 陽一郎) (@takehora) <a href="https://twitter.com/takehora/status/1534082403085750272?ref_src=twsrc%5Etfw">June 7, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

今回は、MWB になれるため+[ISUCON 本](https://gihyo.jp/book/2022/978-4-297-12846-3) の練習に[private-isu](https://github.com/catatsuy/private-isu)を解いていこうと思います。

## 環境構築

AMI で作成していきます。
なんとなく手順を書いておきます。

### 1. AWS コンソール -> EC2 🍇

![ec2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/7a153fb0-ee61-86f5-ca79-4282bcfcbb9e.png)

### 2. Launch Instances 🍈

![ec22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/5954394b-acce-d30a-9873-64bbb5fd20a0.png)

### 3. [private-isu](https://github.com/catatsuy/private-isu) から AMI ID で検索 🍉

![ec23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2a1cddca-8330-c9f9-671b-d76ad9128937.png)

![ec24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/401005c5-21cf-3bcf-b1e1-2d6098a02c84.png)

### 4. Instance type を `c4.large` に 🍊

![ec25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/07cbf344-d4ba-916c-0706-ee2a4fa221f2.png)

### 5. Key pair + セキュリティグループ設定 🍋

![ec26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/0bcc814c-878e-c605-2893-38c11ad0ed97.png)

### 6. ディスクサイズ設定 🍌

![ec27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/85bb94d9-6238-5d11-bc98-e364b6b3c2e7.png)

### 7. 起動 🍎

![ec29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/a00ddee2-a568-b2e7-a575-06cfc3b969fc.png)

![ec212.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/67b03ba9-75ad-6d35-c03a-149f1b52d00d.png)

### 8. SSH 🍍

![ssh.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/e3937146-af1c-cdf8-151f-09c52ebef9f9.png)

![ec211.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/40e2d0ea-7d9d-f2a9-19bd-e8fc2bcc85e7.png)

## MWB 接続 🍏

MWB で、private-isu 内の MySQL に接続します。
![ssh2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/dbb09378-5576-8a4c-a68a-c3de00463708.png)

成功すると、以下のように接続が確立されます。

![ssh3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/4e4f9bc8-0429-3ece-b84c-8df77ffa13df.png)

SQL も実行できます。
![mysqlworkbench3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/477f5dc3-841a-1b2e-1b9f-055a42b3e03b.png)

## ER 図作成 🍐

MWB では、ER 図を簡単に作成できます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/4b4c4a3a-5f1e-d26a-deb6-5820c9392619.png)

![mysqlworkbench4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/53de0a63-1638-f9e1-6bbc-2620e12c8c2b.png)

## スロークエリログ 🍑

`Perfomance Reports` -> `Statement Analysis` から、スロークエリログを確認できます。

![mysqlworkbench5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/ecfb58c8-1150-b74f-764a-319279dc6e7c.png)

```
"SELECT `id`, `user_id`, `body`, `mime`, `created_at` FROM `posts` WHERE `user_id` = ? ORDER BY `created_at` DESC"
```

が N+1 クエリになってそうです。
早速、解消していきましょう。

## private-isu 改善 🍒

![ec230.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/394ad1ec-3c65-5307-803f-3f24ad4aef53.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/410da2e3-24f6-927a-aeef-b5ed79ba4de9.png)

`score: 216` -> `score: 18028`

N+1 クエリと無駄に呼ばれているクエリを解消することで、大幅に改善することができました！

- ここら辺は、[達人が教える Web パフォーマンスチューニング](https://gihyo.jp/book/2022/978-4-297-12846-3)に書いてあるので是非、読んでみて下さい！

## 最後に 🥥

[達人が教える Web パフォーマンスチューニング](https://gihyo.jp/book/2022/978-4-297-12846-3)を読んで、とっても勉強になりました。

良き ISUCON ライフを！
