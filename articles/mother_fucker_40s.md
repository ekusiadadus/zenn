---
title: "『おっさん美少女を描いて！』を；実現したい！"
emoji: "🍇" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: [
"Python",
"機械学習","whisper","StableDiffusion","ChatGPT"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

どうも、おっさんです。
うちの GitHub Copilot の口が悪すぎると話題に！

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/1aa7b376-a5b8-c079-aabc-bbb2a285e439.png)

<!-- <img width="400" alt="うちの GitHub Copilot の口が悪すぎると話題に！" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/1aa7b376-a5b8-c079-aabc-bbb2a285e439.png"> -->

さて、今日は Whisper + Stable Diffusion で永遠の謎『おっさん美少女』を AI に描いて頂こうと思います。
髪の毛は永遠の 0 です。

## 紆余曲折

IntelliJ の PyCharm の YouTube ライブで Jina Cloud が取り上げられていました。
『NewYork にいる Spiderman を描いて！』をしていました。

https://www.youtube.com/watch?v=duWUy5LOEwc

人権がないんです。
家の GPU。

1-2 か月くらい前にかなり Whisper+Stable Diffusion が流行っていたのでやってみたいなという気持ちがありましたが。
Jina Cloud で無料で試せそうだったのでやってみようとして失敗しました...

1. そもそも Jina Cloud のコードが動かない

https://github.com/jina-ai/example-speech-to-image

YouTube のコードは、GitHub 上に公開されいるのですが手順を踏んでも動きません。(2021/12/20)

2. ローカルで動かすと GPU が足りない

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/51950ef1-c8d0-7216-e063-9311438979c6.png)

<!-- <img width="400" alt="ローカルで動かすと GPU が足りない" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/51950ef1-c8d0-7216-e063-9311438979c6.png"> -->

そもそも GPU が足りないので、ローカルで動かすことはできませんでした。

しかし、学習サイズ "medium" や "small" くらいに落とすと動きました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/dc648757-148f-3042-db61-a29a6a8e58fa.png)

<!-- <img width="400" alt="学習サイズ medium や small くらいに落とすと動きました。" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/dc648757-148f-3042-db61-a29a6a8e58fa.png"> -->

3. ui.py が動かない

ここまでくるとボロボロです。
基本的にコードはすべて動きません。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8ad53202-9a73-3d05-4cc0-5f48e3a38bda.png)

<!-- <img width="400" alt="基本的にコードはすべて動きません。" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8ad53202-9a73-3d05-4cc0-5f48e3a38bda.png"> -->

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/5b83df5f-c3d3-9b04-183b-ba318f5c0654.png)

<!-- <img width="400" alt="ここまでくるとボロボロです。" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/5b83df5f-c3d3-9b04-183b-ba318f5c0654.png"> -->

録音したファイルがなぜかグローバルに入っていることになっている...?
ここら辺は、`ffmpeg` 周りのライブラリ問題みたい...

`sudo apt install ffmpeg` で解決しましが、`ui.py` が動かない...
`grpc`周りの接続が、Jina 側に飛ばせない....

https://github.com/jina-ai/dalle-flow/issues/23

まだまだプラットフォーム が未熟で開発中のようで、基本的にコードはすべて動きません。
GPU 強者や、Jina Cloud 詳しい方で成功した人がいれば教えてください。

## (代替品)おっさん美少女 1

https://huggingface.co/spaces/fffiloni/whisper-to-stable-diffusion

ここら辺はデフォルトのモデルです。
Whisper の制度はかなり高いです。(漢字は描いての方を想定していましたが)
正直、日<->英の翻訳から違います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/e3f6476d-f81c-f756-dead-cffc7e046c89.png)

<!-- <img width="400" alt="(代替品)おっさん美少女 1" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/e3f6476d-f81c-f756-dead-cffc7e046c89.png"> -->

`おっさん美少女を書いて` <-> ` Drawing a middle-aged man and a beautiful girl` という感じです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/facda939-e23c-36d0-f927-c64c8d1396e1.png)

<!-- <img width="400" alt="(代替品)おっさん美少女 1-1" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/facda939-e23c-36d0-f927-c64c8d1396e1.png"> -->

無理やり英語を直しても、ダメそうです。
モデルを変えないといけません。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3b5fbadc-e610-ce50-ece9-93bdef41d2ce.png)

<!-- <img width="400" alt="(代替品)おっさん美少女 1-2" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3b5fbadc-e610-ce50-ece9-93bdef41d2ce.png"> -->

## (代替品)おっさん美少女 2 (waifu diffusion)

https://huggingface.co/hakurei/waifu-diffusion?text=%E3%81%8A%E3%81%A3%E3%81%95%E3%82%93%E7%BE%8E%E5%B0%91%E5%A5%B3

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/5a12df6e-d4a0-2f28-4acb-c1a716cce5b7.png)

<!-- <img width="400" alt="(代替品)おっさん美少女 2" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/5a12df6e-d4a0-2f28-4acb-c1a716cce5b7.png"> -->

## (代替品)おっさん美少女 3 (stable diffusion v1.5)

https://huggingface.co/runwayml/stable-diffusion-v1-5?text=%E3%81%8A%E3%81%A3%E3%81%95%E3%82%93%E7%BE%8E%E5%B0%91%E5%A5%B3

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/670c04f2-18f6-439a-5717-749563362a8d.png)

<!-- <img width="400" alt="(代替品)おっさん美少女 3" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/670c04f2-18f6-439a-5717-749563362a8d.png"> -->

## (代替品)おっさん美少女 3 (stable diffusion v2.1)

https://huggingface.co/spaces/stabilityai/stable-diffusion

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/cabe85c7-b594-327b-f8e1-05e8c87f186f.png)

<!-- <img width="400" alt="(代替品)おっさん美少女 3 (stable diffusion v2.1)" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/cabe85c7-b594-327b-f8e1-05e8c87f186f.png"> -->

## 見ると幸せになれるところ

- [BERT/GPT-3/DALL-E 自然言語処理・画像処理・音声処理 人工知能プログラミング実践入門](https://www.amazon.co.jp/kindle-dbs/thankYouPage?_encoding=UTF8&asin=B09DPC4KG7&o=D01-7548446-5177052&a=DT%3AA2CTZ977SKFQZY&isExtendedMarketplace=0&homeMarketplaceId=A1VC38T7YXB528&paymentPlanId=amzn1.pplan.AQ.NA.AAABhSv4M3Q.6JcXb0Nqx-7LrQRWB5Mp5g&paymentContractId=amzn1.pc.puma.YW16bjEucHBsYW4uQVEuTkEuQUFBQmhTdjRNM1EuNkpjWGIwTnF4LTdMclFSV0I1TXA1Zw&redirectionBankName&subtype=STANDARD&eoi=A23ZP02F085DFQ&hasPromotion=false&displayedPrice=3762.0&pointsEarned=38)

[@npaka123](https://twitter.com/npaka123) さんが書かれている本です。
この本は理論的なことをかなり基礎から説明しているガチ勢向けの本だと思っています。
おすすめです。

- [AI とコラボして神絵師になる　論文から読み解く Stable Diffusion 白井暁彦](https://www.amazon.co.jp/AI%E3%81%A8%E3%82%B3%E3%83%A9%E3%83%9C%E3%81%97%E3%81%A6%E7%A5%9E%E7%B5%B5%E5%B8%AB%E3%81%AB%E3%81%AA%E3%82%8B-%E8%AB%96%E6%96%87%E3%81%8B%E3%82%89%E8%AA%AD%E3%81%BF%E8%A7%A3%E3%81%8FStable-Diffusion-%E6%8A%80%E8%A1%93%E3%81%AE%E6%B3%89%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA%EF%BC%88NextPublishing%EF%BC%89-%E7%99%BD%E4%BA%95-%E6%9A%81%E5%BD%A6-ebook/dp/B0BJTVNYNR)

最近の Stable Diffusion モデルを Colab やサンプルコード付きで解説してあります。
理論面も軽く触れています。
個人的に、クリエイターが AI とどのように折り合いをつけるかに章がさかれていて凄く面白かったです。

- [【簡単】ローカル環境で stable-diffusion で実行する方法](https://self-development.info/%E3%80%90%E7%B0%A1%E5%8D%98%E3%80%91%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E7%92%B0%E5%A2%83%E3%81%A7stable-diffusion%E3%81%A7%E5%AE%9F%E8%A1%8C%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95/)

そもそも、機械学習全然詳しくないのでここら辺をちらちら見ながらやっています。

## まとめ

年末年始で自分のモデルを作っていこうという気持ち

## 雑談

最近飼っているうちの AI 達です。

- GitHub Copilot (年: $100)
- ChatGPT (月: $6)
- Stable Diffusion (月: $10)

https://huggingface.co/spaces/fffiloni/whisper-to-stable-diffusion

高い...
5 年後には自分の仕事なくなって欲しいですね。
Whisper + ChatGPT とか組み合わせ無限大！という感じですね。

年末で 10 連飲み会が発生しているので美少女に救われたい。
おっさんは帰れ！
