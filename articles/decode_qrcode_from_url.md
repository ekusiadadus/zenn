---
title: QRコードをURLからデコードするアプリをFlutterで作る
tags: Flutter, QRCode, Dart
author: ekusiadadus
slide: false
---

# QR コードを URL からデコードするアプリを Flutter で作る

こんにちは、@ekusiadadus です。
ひょんなことから QR コードを URL や Image ファイルからデコードするスマホアプリを作ることになりました。
いろいろ調べてみたのですが、なかなか文献が見当たらなかったので自分で作ってみました。

## 作ったアプリ

ソースコード: https://github.com/ekusiadadus/flutter-barcode-decode-from-url

![205739914-d9e65c78-f1f5-476b-b6c9-56bedccd779e.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/655bb3f2-b461-7bf3-18fb-94f8991b5d48.gif)

## 作り方

いろいろ考えたのですが、一番シンプルなのは次の手順だと思いました。

1. URL から画像を一時ストレージに保存させる
2. 画像ファイルから scan パッケージで、QR コードを読み取る

### 1. URL から画像を一時ストレージに保存させる

まずは、URL から画像を一時ストレージに保存させます。
本当はメモリ上にのせておくだけで読み取れると思うのですが、scan パッケージがファイルを読み取るようなので、一時ファイルに保存させます。
(もしやり方わかる方いたら教えてください)

```dart
final qrCodeController = TextEditingController();

//Image is a temporary memory Image
Image? image;
//File is a temporary file
File? imageFile;

void onGetImage(String url) async {
  final tmpFile = await urlToFile(url);
  setState(() {
    imageFile = tmpFile;
    image = Image.file(tmpFile);
  });
}
```

urlToFile は、次のように実装しました。
一時ファイルなので秒単位でファイル名が変わるようにしました。

```dart
// url to file function
Future<File> urlToFile(String imageUrl) async {
  Directory tempDir = await getTemporaryDirectory();
  String tempPath = tempDir.path;
  //datetime to make the file name unique
  File file = File('$tempPath${DateTime.now()}.png');
  http.Response response = await http.get(Uri.parse(imageUrl));
  await file.writeAsBytes(response.bodyBytes);

  return file;
}
```

## 画像ファイルから scan パッケージで、QR コードを読み取る

次に、画像ファイルから scan パッケージで、QR コードを読み取ります。
こんな感じで、scan パッケージの parse メソッドを使います。

```dart
//Result of the QR code
String? result;

void onGetImage(String url) async {
  final tmpFile = await urlToFile(url);
  final qr = await Scan.parse(tmpFile.path);
  setState(() {
    imageFile = tmpFile;
    image = Image.file(tmpFile);
    result = qr;
  });
}
```

state が更新されたら、画像が表示されるようにします。

```dart

body: Center(
  child: Column(
    mainAxisAlignment: MainAxisAlignment.center,
    children: <Widget>[
      TextField(
        controller: qrCodeController,
        decoration: InputDecoration(
          border: OutlineInputBorder(),
          labelText: 'QR Code URL',
          suffixIcon: qrCodeController.text.isNotEmpty
              ? IconButton(
                  icon: const Icon(Icons.clear),
                  onPressed: () {
                    qrCodeController.clear();
                  },
                )
              : Container(width: 0),
        ),
      ),
      const SizedBox(height: 20),
      image ?? Container(),
      const SizedBox(height: 20),
      Text(result ?? ''),
    ],
  ),
),

```

![barcode_from_url_tutorial – main.dart [barcode_from_url_tutorial] 2022-12-07 06-04-42.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/fd65637d-16a8-03ed-1b4c-6d82bf0ea279.gif)

## まとめ

Flutter で QR コードを URL や Image ファイルからデコードするアプリを作る方法を紹介しました。
scan パッケージを使うと、簡単に実装できました。
他にも、qr_code_flutter とか google ml kit 使う方法があると思います。

オンメモリでやる方法ない気がするんだよなぁ...
