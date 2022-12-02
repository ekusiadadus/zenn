---
title: GCP Cloud Vision よく使う機能まとめ ~ Go言語 ~
tags: GCP, Go,
author: ekusiadadus
slide: false
---

# GCP Cloud Vision 機能一覧 with Go 言語

『画像や動画から、文字情報やどのような物体が映っているかを AI で抜き出したい！』と思っている、そこのあなた
[Cloud Vision](https://cloud.google.com/vision) していますか？

Google Cloud Platform の [Cloud Vision](https://cloud.google.com/vision) というサービスを使うと簡単に画像認識 OCR や物体検知を高性能で体験できます。
Cloud Vision API の機能一覧と実際にどのような場面で使えるかをまとめました。

[Google Cloud Platform](https://github.com/GoogleCloudPlatform) が[豊富な例を GitHub](https://github.com/GoogleCloudPlatform/golang-samples)に上げてくれているので、興味ある方は実際に触ってみてください。

## 機能一覧

Cloud Vision API は以下の機能を提供しています。(他にもあるかも)

- [GCP Cloud Vision 機能一覧 with Go 言語](#gcp-cloud-vision-機能一覧-with-go-言語)
  - [機能一覧](#機能一覧)
  - [顔認識](#顔認識)
  - [ラベル検出](#ラベル検出)
  - [ランドマーク検出](#ランドマーク検出)
  - [テキスト検出](#テキスト検出)
  - [ドキュメントテキスト検出](#ドキュメントテキスト検出)
  - [プロパティ検出](#プロパティ検出)
  - [Web 検出](#web-検出)
  - [会社のロゴで全部やってみる](#会社のロゴで全部やってみる)
  - [最後に](#最後に)

## 顔認識

GCP ページ: [Detecting Faces](https://cloud.google.com/vision/docs/detecting-faces)

人の顔が映っていた場合、顔写真のプロパティを出力します。

例えば以下のような類推をしてくれます。

- 上下、左右の顔の向き
- 喜怒哀楽などの感情の類推
- 顔のパーツが写真のどの位置に存在するか

```go
  client, err := vision.NewImageAnnotatorClient(ctx)
  // anotations には、顔認識の結果が入っている
  // func (c *ImageAnnotatorClient) DetectFaces(ctx context.Context, img *pb.Image, ictx *pb.ImageContext, maxResults int, opts ...gax.CallOption) ([]*pb.FaceAnnotation, error)
  annotations, err := client.DetectFaces(ctx, image, nil, 10)
```

![test.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/36ca8ed3-20a0-4ef4-814c-55f972889344.png)

```
Faces:
  Face 0
    Anger: VERY_UNLIKELY
    Joy: LIKELY
    Surprise: VERY_UNLIKELY
		Sorrow: VERY_UNLIKELY
    Headwear: VERY_UNLIKELY
    Blurred: VERY_UNLIKELY
    Under-exposed: VERY_UNLIKELY
    FdBoundingPoly: vertices:{x:1209 y:1757} vertices:{x:1664 y:1757} vertices:{x:1664 y:2204} vertices:{x:1209 y:2204}
    FaceLandmarks:
 [type:LEFT_EYE position:{x:1382.8287 y:1930.6954 z:0.0009727478} type:RIGHT_EYE position:{x:1408.4132 y:1961.6671 z:-109.46489} type:LEFT_OF_LEFT_EYEBROW position:{x:1373.89 y:1875.0829 z:31.537819} type:RIGHT_OF_LEFT_EYEBROW position:{x:1384.1958 y:1892.9578 z:-50.71999} type:LEFT_OF_RIGHT_EYEBROW position:{x:1387.2249 y:1922.5789 z:-100.2508} type:RIGHT_OF_RIGHT_EYEBROW position:{x:1439.4751 y:1953.7578 z:-155.33568} type:MIDPOINT_BETWEEN_EYES position:{x:1370.877 y:1939.9951 z:-63.574837} type:NOSE_TIP position:{x:1326.5204 y:2000.4471 z:-49.927307} type:UPPER_LIP position:{x:1345.025 y:2056.598 z:-23.065115} type:LOWER_LIP position:{x:1357.4213 y:2071.9465 z:-13.0831585} type:MOUTH_LEFT position:{x:1361.0638 y:2029.415 z:39.39473} type:MOUTH_RIGHT position:{x:1383.495 y:2089.0693 z:-51.860104} type:MOUTH_CENTER position:{x:1345.3228 y:2057.8105 z:-15.859501} type:NOSE_BOTTOM_RIGHT position:{x:1382.8903 y:2033.4562 z:-63.039738} type:NOSE_BOTTOM_LEFT position:{x:1346.0057 y:2000.881 z:1.6753235} type:NOSE_BOTTOM_CENTER position:{x:1348.9897 y:2031.4204 z:-32.725193} type:LEFT_EYE_TOP_BOUNDARY position:{x:1382.684 y:1920.6149 z:-5.499584} type:LEFT_EYE_RIGHT_CORNER position:{x:1391.7933 y:1935.773 z:-22.60736} type:LEFT_EYE_BOTTOM_BOUNDARY position:{x:1376.575 y:1939.8978 z:5.9174004} type:LEFT_EYE_LEFT_CORNER position:{x:1377.663 y:1924.4435 z:28.603077} type:RIGHT_EYE_TOP_BOUNDARY position:{x:1410.4309 y:1952.8815 z:-118.572784} type:RIGHT_EYE_RIGHT_CORNER position:{x:1430.7496 y:1972.6544 z:-130.17412} type:RIGHT_EYE_BOTTOM_BOUNDARY position:{x:1406.6226 y:1973.8807 z:-107.85469} type:RIGHT_EYE_LEFT_CORNER position:{x:1392.5935 y:1955.4607 z:-84.61473} type:LEFT_EYEBROW_UPPER_MIDPOINT position:{x:1380.1323 y:1873.5831 z:-16.011051} type:RIGHT_EYEBROW_UPPER_MIDPOINT position:{x:1408.8989 y:1924.9741 z:-134.85135} type:LEFT_EAR_TRAGION position:{x:1520.012 y:1926.5598 z:127.898605} type:RIGHT_EAR_TRAGION position:{x:1625.7051 y:2128.3757 z:-111.01236} type:FOREHEAD_GLABELLA position:{x:1377.934 y:1912.875 z:-75.16867} type:CHIN_GNATHION position:{x:1344.6588 y:2127.853 z:16.881191} type:CHIN_LEFT_GONION position:{x:1422.3843 y:2040.4146 z:131.25916} type:CHIN_RIGHT_GONION position:{x:1480.9247 y:2137.9443 z:-72.10258} type:LEFT_CHEEK_CENTER position:{x:1365.6957 y:1989.725 z:55.372932} type:RIGHT_CHEEK_CENTER position:{x:1414.894 y:2046.1897 z:-99.02522}]
```

複数人映っていても判定してくれます。

```
Faces:
  Face 0
    Anger: VERY_UNLIKELY
    Joy: UNLIKELY
    Surprise: VERY_UNLIKELY
  Face 1
    Anger: VERY_UNLIKELY
    Joy: VERY_UNLIKELY
    Surprise: VERY_UNLIKELY
  Face 2
    Anger: VERY_UNLIKELY
    Joy: VERY_UNLIKELY
    Surprise: VERY_UNLIKELY
  Face 3
    Anger: VERY_UNLIKELY
    Joy: VERY_UNLIKELY
    Surprise: VERY_UNLIKELY
  Face 4
    Anger: VERY_UNLIKELY
    Joy: VERY_UNLIKELY
    Surprise: VERY_UNLIKELY
```

## ラベル検出

GCP ページ: [ラベル検出](https://cloud.google.com/vision/docs/label-detection?hl=ja)

画像に写っている物体を検出してくれます。

![shokki.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/f14b0e75-10b7-732e-c3e7-5a514904c4a7.jpeg)

```go
  client, err := vision.NewImageAnnotatorClient(ctx)
  // anotations には 物体検出 が入っている
  //func (c *ImageAnnotatorClient) DetectLabels(ctx context.Context, img *pb.Image, ictx *pb.ImageContext, maxResults int, opts ...gax.CallOption) ([]*pb.EntityAnnotation, error)
  annotations, err := client.DetectLabels(ctx, image, nil, 10)
```

```
Labels:
Shelf
Tableware
Shelving
Wood
Drinkware
Gas
Serveware
Hardwood
Machine
Room
```

## ランドマーク検出

GCP ページ: [ランドマーク検出](https://cloud.google.com/vision/docs/landmark-detection?hl=ja)

画像に写っているランドマークを検出してくれます。

```go
  client, err := vision.NewImageAnnotatorClient(ctx)
  // anotations には ランドマーク検出 が入っている
  //func (c *ImageAnnotatorClient) DetectLandmarks(ctx context.Context, img *pb.Image, ictx *pb.ImageContext, opts ...gax.CallOption) ([]*pb.EntityAnnotation, error)
  annotations, err := client.DetectLandmarks(ctx, image, nil)
```

![oosakajou.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/6dc33c22-72af-bf06-f5a4-d86bf6f8bc1b.jpeg)

```
Landmarks:
Osaka Castle Park
```

## テキスト検出

GCP ページ: [テキスト検出](https://cloud.google.com/vision/docs/text-detection?hl=ja)

画像に写っているテキストを検出してくれます。
しかもなんとヘブライ語対応！

```go
  client, err := vision.NewImageAnnotatorClient(ctx)
  // anotations には テキスト検出 が入っている
  //func (c *ImageAnnotatorClient) DetectTexts(ctx context.Context, img *pb.Image, ictx *pb.ImageContext, opts ...gax.CallOption) ([]*pb.EntityAnnotation, error)
  annotations, err := client.DetectTexts(ctx, image, nil)
```

![hebrew.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/d23fe239-e455-7f54-20a7-175769e4dd91.jpeg)

```
Text:
"שְׁמַע יִשְׂרָאֵל יְהוָה אֱלֹהֵינוּ\nן\nיְהוָה אֱהָדֹ: וְאָהַבְתָּ אֵת\nיְהוָה אֱלֹהֶיךָ בְּכָל־לְבָבְךָ\nוּבְכָל־נַפְשְׁךָ וּבְכָל־מְאֹדֶךָ\nAFE J857\n*\n:\n,\nT\n[40 34\n-3"
"שְׁמַע"
"יִשְׂרָאֵל"
"יְהוָה"
"אֱלֹהֵינוּ"
"ן"
"יְהוָה"
"אֱהָדֹ"
":"
"וְאָהַבְתָּ"
"אֵת"
"יְהוָה"
"אֱלֹהֶיךָ"
"בְּכָל־לְבָבְךָ"
"וּבְכָל־נַפְשְׁךָ"
"וּבְכָל־מְאֹדֶךָ"
"AFE"
"J857"
"*"
":"
","
"T"
"["
"40"
"34"
"-3"
```

## ドキュメントテキスト検出

テキスト検出のもう少し詳しいバージョンだと思っています。

![doctext.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/b5ce4d5f-bd51-1510-dd75-641f7b2bfd17.png)

```
--- detectDocumentText
Document Text:
"テキスト検出\n画像に写っているテキストを検出してくれます。\nしかもなんとヘブライ語対応!\nclient, err := vision.NewImageAnnotatorClient(ctx)\n// anotations にはテキスト検出が入っている\n//func (c Image AnnotatorClient) DetectTexts (ctx context.Context, img *pb.Image, ic\nannotations, err := client.DetectTexts(ctx, image, nil)"
Pages:
	Confidence: 0.988716, Width: 772, Height: 386
	Blocks:
		Confidence: 0.989059, Block type: TEXT
		Paragraphs:
			Confidence: 0.989059			Words:
				Confidence: 0.990139, Symbols: テキスト
				Confidence: 0.986898, Symbols: 検出
		Confidence: 0.990634, Block type: TEXT
		Paragraphs:
			Confidence: 0.992576			Words:
				Confidence: 0.992444, Symbols: 画像
				Confidence: 0.995960, Symbols: に
				Confidence: 0.996960, Symbols: 写っ
				Confidence: 0.996308, Symbols: て
				Confidence: 0.991980, Symbols: いる
				Confidence: 0.995734, Symbols: テキスト
				Confidence: 0.994331, Symbols: を
				Confidence: 0.994469, Symbols: 検出
				Confidence: 0.995566, Symbols: し
				Confidence: 0.997074, Symbols: て
				Confidence: 0.997135, Symbols: くれ
				Confidence: 0.992679, Symbols: ます
				Confidence: 0.943153, Symbols: 。
			Confidence: 0.987583			Words:
				Confidence: 0.989595, Symbols: しかも
				Confidence: 0.987171, Symbols: なんと
				Confidence: 0.984599, Symbols: ヘブライ
				Confidence: 0.991614, Symbols: 語
				Confidence: 0.992737, Symbols: 対応
				Confidence: 0.980377, Symbols: !
		Confidence: 0.986455, Block type: TEXT
		Paragraphs:
			Confidence: 0.983791			Words:
				Confidence: 0.987956, Symbols: client
				Confidence: 0.997707, Symbols: ,
				Confidence: 0.992329, Symbols: err
				Confidence: 0.986108, Symbols: :
```

## プロパティ検出

GCP ページ: [プロパティ検出](https://cloud.google.com/vision/docs/detecting-properties?hl=ja)

画像の色や明るさなどのプロパティを検出してくれます。
(ドミナントカラー以外の使い方をわかっていません...)

![image-qiita_brand_color.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/6bf27cb4-44bd-1cca-8c98-133815a29c9c.png)

```go
  client, err := vision.NewImageAnnotatorClient(ctx)
  // anotations には 画像プロパティ が入っている
  //func (c *ImageAnnotatorClient) DetectImageProperties(ctx context.Context, img *pb.Image, ictx *pb.ImageContext, opts ...gax.CallOption) (*pb.ImageProperties, error)
  annotations, err := client.DetectImageProperties(ctx, image, nil)
```

```
Dominant colors:
18.9% - #55c500
68.1% - #f7f7f7
0.2% - #bde89c
2.2% - #363939
0.5% - #e1f5d1
1.5% - #9a9b9b
2.2% - #4dc200
1.0% - #70ce2a
0.3% - #87d64c
0.2% - #a0df70
```

## Web 検出

GCP ページ: [Web 検出](https://cloud.google.com/vision/docs/detecting-web?hl=ja)

画像を Web で検索し、類似の画像を含む URL を返してくれます。

```go
  client, err := vision.NewImageAnnotatorClient(ctx)
  // anotations には Web 検出 が入っている
  //func (c *ImageAnnotatorClient) WebDetection(ctx context.Context, img *pb.Image, ictx *pb.ImageContext, opts ...gax.CallOption) (*pb.WebDetection, error)
  annotations, err := client.WebDetection(ctx, image, nil)
```

![banksey.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/11569e95-f1d6-a002-b1eb-256741832853.jpeg)

```
Web properties:
	Pages with this image:
		https://www.pinterest.com/schulz0247/malibu-real-estate/
		https://www.phillips.com/artist/8845/banksy
		https://www.amazon.com/Banksy/s?k=Banksy&page=2
		https://www.pinterest.com/pin/214695107213181526/
		https://www.etsy.com/hk-en/listing/1073717407/girl-with-red-balloon-wall-decal
		https://www.etsy.com/in-en/listing/1073717407/girl-with-red-balloon-wall-decal
		https://twitter.com/seahawks/status/1595548973603094530?lang=en
		https://www.amazon.com/Banksy-Canvas-Wall-Art-12x16inch/dp/B08J287S84
		https://www.amazon.com/Balloon-Graffiti-Painting-20x30cm-Unframed/dp/B09DGD3PHM
		https://in.pinterest.com/pin/661184789021752697/
	Entities:
		Entity		Score	Description
		/g/11bxdrtkl2 	1.0714	Balloon Girl
		/g/11s86dc227 	1.0560	Banksy
		/m/0jjw       	0.7040	Art
		/m/05qdh      	0.7012	Painting
		/m/034wh      	0.6947	Graffiti
		/m/01n5jq     	0.6680	Poster
		/m/07vwy6     	0.6231	Street art
		/m/0n1h       	0.5725	Artist
		/m/0h0vk      	0.5155	Contemporary art
		/m/0jg24      	0.4971	Image
	Best guess labels:
		bristol arms hotel
```

## 会社のロゴで全部やってみる

GCP ページ: [ロゴ検出](https://cloud.google.com/vision/docs/detecting-logos?hl=ja)

```go
  client, err := vision.NewImageAnnotatorClient(ctx)
  // anotations には ロゴ検出 が入っている
  //func (c *ImageAnnotatorClient) DetectLogos(ctx context.Context, img *pb.Image, ictx *pb.ImageContext, opts ...gax.CallOption) ([]*pb.EntityAnnotation, error)
  annotations, err := client.DetectLogos(ctx, image, nil)
```

![matsurilogoofficial.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/e106f712-b173-ff07-d482-532e70c62553.png)

```
--- detectFaces
No faces found.
--- detectLabels
Labels:
Font
Brand
Logo
Graphics
Magenta
Symbol
Circle
Trademark
--- detectLandmarks
No landmarks found.
--- detectText
Text:
"സ\nmatsuri\ntechnologies"
"സ"
"matsuri"
"technologies"
--- detectDocumentText
Document Text:
"സ\nmatsuri\ntechnologies"
Pages:
	Confidence: 0.564514, Width: 1224, Height: 270
	Blocks:
		Confidence: 0.138816, Block type: TEXT
		Paragraphs:
			Confidence: 0.138816			Words:
				Confidence: 0.138816, Symbols: സ
		Confidence: 0.990213, Block type: TEXT
		Paragraphs:
			Confidence: 0.990213			Words:
				Confidence: 0.988272, Symbols: matsuri
				Confidence: 0.991346, Symbols: technologies
--- detectLogos
Logos:
IMS Learning Resources
--- detectProperties
Dominant colors:
11.7% - #ed2224
2.9% - #f79c9e
3.0% - #fcd7d7
1.1% - #f25e60
2.7% - #f03f41
2.4% - #f36c6e
1.6% - #f68f91
0.8% - #f9b5b6
1.1% - #fac0c1
0.2% - #ea0000
--- detectCropHints
Crop hints:
(373,0)
(850,0)
(850,269)
(373,269)
--- detectWeb
Web properties:
	Full image matches:
		https://gsimg.asiayo.com/ay-image-upload/1615537371438_line_oa_chat_210310_164432.jpg
		https://prtimes.jp/i/76057/80/ogp/d76057-80-23015500ed05ed4dc610-0.png
		https://i0.wp.com/moneyzone.jp/wp-content/uploads/2022/03/2022-03-25_10-09-58_911535.jpeg?fit=1224%2C270&quality=100&strip=all&resize=100&ssl=1
		https://prcdn.freetls.fastly.net/release_image/76057/80/76057-80-5da5016bf2da76d1435f0087e0e16ccf-1224x270.png?format=jpeg&auto=webp&quality=85%2C75&width=1950&height=1350&fit=bounds
		https://www.matsuri.tech/_nuxt/img/matsurilogoofficial.70b766f.png
		https://residencetokyo.com/jp/wp-content/uploads/2020/03/matsuri.png
		https://findy-code-images.s3.ap-northeast-1.amazonaws.com/companies/00731_matsuritechnologies.png
		https://static.wixstatic.com/media/88d34e_eac6b406a99c43e1a2323660e563b950~mv2.png/v1/fill/w_664,h_146,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/matsurilogoofficial.png
		https://static.wixstatic.com/media/88d34e_eac6b406a99c43e1a2323660e563b950~mv2.png/v1/fill/w_658,h_144,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/88d34e_eac6b406a99c43e1a2323660e563b950~mv2.png
		https://prtimes.jp/i/22329/836/resize/d22329-836-81ad96c43580f0e46fd4-1.png
	Entities:
		Entity		Score	Description
		/g/11f6fzyh_1 	0.5706	matsuri technologies ㈱
		/m/0dwx7      	0.3678	Logo
		/m/0rzqv1j    	0.3330
		/g/120y8l81   	0.3036	Enterprise
		/m/03g09t     	0.2674	Clip art
		/g/120z18l3   	0.2385	Company
		/m/01_jrm     	0.2281	Joint-stock company
		/m/01cd9      	0.2236	Brand
		/m/023k2      	0.1891	Corporation
		/m/03jzl9     	0.1738	Share
	Best guess labels:
		matsuri technologies 株式 会社
--- detectWebGeo
Entities:
	Entity		Score	Description
	/g/11f6fzyh_1 	0.5706	matsuri technologies ㈱
	/m/0dwx7      	0.3678	Logo
	/m/0rzqv1j    	0.3341
	/g/120y8l81   	0.3036	Enterprise
	/m/03g09t     	0.2674	Clip art
	/g/120z18l3   	0.2322	Company
	/m/01cd9      	0.2236	Brand
	/m/01_jrm     	0.2207	Joint-stock company
	/m/023k2      	0.1886	Corporation
	/m/03jzl9     	0.1709	Share
--- detectSafeSearch
Safe Search properties:
Adult: VERY_UNLIKELY
Medical: UNLIKELY
Racy: VERY_UNLIKELY
Spoofed: VERY_UNLIKELY
Violence: UNLIKELY
--- localizeObjects
No objects found.
```

## 最後に

今回は、Cloud Vision API を使って画像の解析を行ってみました。
画像データから、テキストやロゴ、顔、物体、セーフサーチ、Web 情報などを取得することができます。

実際に matsuri technologies では物件の写真から、備品の情報や清掃状況などを取得しています。(GCP Cloud Vison + AWS Rekognition)
非常にお手頃価格で、高精度な画像解析ができるので、ぜひ活用してみてください。
