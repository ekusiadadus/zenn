---
title: クラウドサーバーを作る
tags: RaspberryPi nextcloud ubuntu20.04
author: ekusiadadus
slide: false
---
# クラウドサーバーを作る
この記事では、Raspberry Pi4、NextCloudを使ってクラウドサーバーを作成します。
## 環境

|環境 |
|---|---|
|Raspberry Pi 4 modelB|
|Ubuntu 20.04  |
|NextCloud |
## §1.Raspberry Piをセットアップする
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/e533a117-1679-5d87-2eeb-1a128ced85f8.png)
### §1.1Ubuntuをインストールする
Raspberry Pi Imagerを使って、Ubuntu LTS 2020をMicorSDに入れます。
Imagerで、フォーマット等ができます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/016b444d-3bb8-c5e0-1c4e-1c600168cddb.png)


**フォーマット**
(Rasberry Pi Imagerを使うと、フォーマットもしてくれます)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2a66700d-c1ff-f4cf-3f2c-c90f8e284ccc.png)



### §1.2Ubuntuを起動する
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/01ec1fe5-7d22-30ad-1639-61a9e77ce564.png)


ラズパイ4はデスクトップ向けUbuntuでもサクサク動きます。

### §1.3アップデート
![raspi2020126_4.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/07b64777-13ff-b627-5626-8c793c38348f.gif)


## §2.クラウド構築
### §2.1 NextCloudインストール
```
sudo snap install nextcloud
```
![raspi10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/ddb9336e-d39d-8695-0c82-11b7d7e01805.png)

### §2.2 管理アカウント設定
```
sudo nextcloud.manual-install username password
```
## §3.使ってみる
Raspberry PiのIPアドレスを確認して、同じネットワーク内にある別PCからwebブラウザで見るとNext Cloudを使用できるようになります。
```
http://ラズパイのIPアドレス
```
Raspberry PiのIPアドレスを固定したり、ドメインを設定することで外部からでもクラウドサーバーに接続できるようになります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/35acb465-836d-14b4-6ce0-947e3efa093a.png)

## §4.おまけ(ファイルサーバーを作る)

### Sambaインストール
```
sudo apt install samba samba-common-bin
```

### Sambaのconfigを変える
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2df3c2f8-080c-6288-4999-0fbc322dbb86.png)
```
[share]
path = /home/share
browseable = yes
writable = yes
省略
```
### Shareフォルダを作る
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/4ffebd80-b908-ba04-b016-da142fcca54c.png)


### Samba再起動
```
sudo systemctl restart samba
```

## §5.おまけ(ラズパイにssh接続(Windows))
```
ssh RaspberryPiユーザ名@IPアドレス
```
## §5.終わりに
いかがでしたでしょうか？
Raspberry Piを使うと、自宅でサーバー周りについて、手を動かしながら学べるのでとても楽しいです。

