---
title: "GitHub Copilot for CLI があればコマンドなんてもう覚えなくていいのでは？" # 記事のタイトル
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["GitHub", "Copilot", "ChatGPT", "OpenAI", "CopilotforCLI"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# GitHub Copilot for CLI があればコマンドなんてもう覚えなくていいのでは？

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/e370e49d-7f97-32a3-8b21-64fa87ef9298.png)

どうも @ekusiadadus です。
なぜか、ウェイティングリストの最初をもらうことが多いです <- 本当に謎
OpenAI の GPT-4 とか 今回の GitHub Copilot とか
その分、レビューして貢献していこうと思います

今回は、GitHub Copilot for CLI に関してです
[公式](https://githubnext.com/projects/copilot-cli/) によると

> シェルのコマンドやフラグを覚えるのに苦労したことはないだろうか。シェルにやってほしいことをそのまま言えたらいいのに、と思ったことは？GitHub Copilot のアシスタンスをターミナルに組み込むことで、そんな心配はもういらない。 (意訳)

ということで、CLI で GitHub Copilot を使えるようになりました。

GitHub Copilot for CLI は、GitHub Copilot と同じく、AI がコマンドを生成してくれるものです。

コマンドは、**3 つ**です

## 使い方

1. `??`
2. `git?`
3. `gh?`

## `??`

このコマンドは、**シェルの任意のコマンドを AI に聞くこと**ができます。

**公式**

1. TypeScript のファイルを検索したい
1. node_modules の中身は無視したい
1. サイズでソートをかけたい

![copilotcli1.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8d281c26-cb0a-5f7c-b979-8b5e0e59b5a2.gif)

**やってみた**

1. Go 言語をインストールしたい
1. Go 言語のバージョンは、1.21 にしたい
1. apt-get でなく、wget を使いたい

![copilotcli4.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/4d085c2f-95ad-5c48-fcc7-0eb92a059bdb.gif)

少し意地悪をして、まだリリースされていない`go1.21` をインストールしたいというコマンドを AI に聞いてみました。

上の gif をみると、go1.16.3 を進めてきたり、apt-get を使わないでという指示を無視するので、まだ完全ではありませんが、言語やアプリをインストールする際に、Google に聞く必要はなくなりそうです。

## `git?`

`git?`は、git の起動に特化した検索に使用されます。`git?`と比べると、git コマンドを生成する能力が高く、git のコンテキストであることを説明する必要がない場合は、クエリーをより簡潔にすることができます。

とのことです。
おそらく内部的には、`??` と同じで事前に git コマンドであるという学習をさせているのではないかと思います。
もしかしたら、学習データを git コマンドに特化させているのかもしれません。(未確認)

**公式**

1. ブランチを削除したい
2. `test-branch-1`を削除したい

![copilotcli2.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/5ae05c33-8af1-631f-33ba-89275b754ed1.gif)

**やってみた**

1. main ブランチ を deploy ブランチにマージしたい
2. main ブランチのいくつかのコミットのみをマージしたい

![copilotcli5.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8829b528-0d8b-819b-1b09-d93783c3cefd.gif)

上のような感じで、日本語でも答えてくれますが前後の文脈を読んでくれているのかは微妙だと思います。
単純にプロンプトをつなげているだけのようにも見えます。

## `gh?`

`gh`は、GitHub CLI のコマンドとクエリのインターフェイスのパワーと、AI が複雑なフラグと `jq` を生成してくれる便利さを兼ね備えています。

**公式**

1. vercel/next.js のリポジトリの issue を開きたい
2. shell 上で開きますか？

![copilotcli3.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/ef796c36-8585-809a-2905-90495cb83ccb.gif)

**やってみた**

1. GitHub で assign されている PR を全て列挙する
2. GitHub の PR ページを開く

![copilotcli6.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3dc57eb6-c8a0-37b6-c4f5-26404bb5cdc1.gif)

WSL からやったので、WSL GUI 　が出てしまいましたが、良さげ

ただ、ちゃんと聞かないとたまにやばいコマンドを打ち出すことがあるので気を付けてください。

![copilotcli7.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/04301bc3-235c-d56c-12ff-ee885910ccf3.gif)

## インストールはこちらから

https://www.npmjs.com/package/@githubnext/github-copilot-cli
