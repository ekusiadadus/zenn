---
layout: ../../layouts/MarkdownPostLayout.astro
title: "WSL2 でホットリロードが効かない問題の解決方法"
pubDate: 2022-02-01
description: "WSL2 でホットリロードが効かない問題の解決方法"
author: "@ekusiadadus"
image:
  url: "https://avatars.githubusercontent.com/u/70436490?s=400&u=a714da7802c65046265c6848887eecddfc58b5c0&v=4"
  alt: "WSL2 でホットリロードが効かない問題の解決方法"
tags: ["wsl2", "hot-reload", "windows", "yarn", "node.js"]
---

# WSL2 でホットリロードが効かない問題の解決方法

先日から、WSL2 でホットリロードが効かない問題が発生していました。
主にフロントエンドの開発時に、結構困っていて、解決方法を探していました。
WSL2 側の google-chrome GUI を使えば、問題なくホットリロードできるのですが、Windows 側のブラウザで確認したい場合に、ホットリロードができずに困っていました。

## 問題原因

問題の原因は、WSL2 で Windows の環境変数を読みに行っていたことが問題でした。
たまたま、Flutter 関連の環境構築でエラーが出ていて、その時に、WSL2 側の環境変数を確認したところ、Windows 側の環境変数が読み込まれていました。

`flutter doctor` でエラーが出ていたので、WSL2 側の環境変数を確認してみました。

```
/usr/bin/env: ‘bash\r’: No such file or directory
```

これをみて、そんなはずないのになぁと環境変数を確認していた時に発覚しました。

## 解決方法

Windows の環境変数を読みに行かないようにすることで、解決できました。

WSL2 側の、`/etc/wsl.conf` に以下の設定を追加することで、Windows の環境変数を読みに行かないようにできます。

```
[interop]
appendWindowsPath = false
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/d401132f-6060-2121-0fd3-95f555400579.png)

↑ 解決する前は、Windows の環境変数を優先していた。
