---
title: "IDEショートカットは怖くない"
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["goland", "CLion", "Pycharm", "IntelliJ", "VSCode"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

こんにちは、[@ekusiadadus](https://qiita.com/ekusiadadus)です。

IntelliJ を本格的に業務で使いだして、半年が経ちました。
最初は、IntelliJ 使いづらい...となっていましたが、今では、IntelliJ の方が速く実装できる場面が多くなりました。

業務で Goland, PyCharm、趣味で Clion を使っています。
IntelliJ 関連でよく出る便利わざを VSCode と比較しながらご紹介します。

まず最初に、**ショートカットの変更は全く怖くない**ということをお伝えします。
自分の知り合いや、後輩にショートカットを変更するのが怖いと言われることがあります...

よく使う機能にはショートカットを割り当てることで、より効率的に作業ができます。

ショートカットはすぐにデフォルトに戻せるので、恐れずにどんどん自分の使いやすいショートカットを割り当てていきましょう。

(力尽きたので、時間があるときに追記します。たぶん。)

## IntelliJ ショートカットの変更方法

1. `File` -> `Settings` (`Ctrl + Alt + S`)
   ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/686a70e4-5068-bae9-93d8-5bd4b31f24cd.png)

2. `Keymap` -> 追加したい機能に対して`Add keyboard Shortcut` をする
   ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/acf7ba15-b6b7-3a4f-814b-47c8449c176b.png)

3. 実際に加えたいショートカットを打ち込む
   ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/56feaedb-fbc4-0206-b84e-3c0eb337ccb5.png)

の 3 つの手順を踏めばすぐに変更できます。

## VSCode ショートカットの変更方法

1. `Ctrl + Shift + P`（Windows の場合）または `Command + Shift + P`（macOS の場合）を押して、コマンドパレットを開きます。
   ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/f7e7da43-ce43-73a7-0b07-bc8b0acbc737.png)

2. コマンドパレットで、「環境設定」と入力します。キーボードショートカットを開く」と入力し、Enter キーを押します。
   ![qiita10.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/f025ef54-b9b8-113d-a206-da3e254cf908.gif)

3. エディタで keybindings.json ファイルが開かれます。
   keybindings.json ファイルには、利用可能なすべてのショートカットとそれに対応するアクションのリストが表示されます。
   このファイルを変更しましょう。

### ショートカット一覧

**文字編集関連**

| 機能                             | IntelliJ                 | VSCode                   |
| :------------------------------- | :----------------------- | :----------------------- |
| コメントアウト                   | `Ctrl + /`               | `Ctrl + /`               |
| 同じ変数を一括更新               | `Ctrl + Alt + V` or `F2` | `Shift + F6`             |
| 行の入れ替え                     | `Ctrl + Shift + ↑↓`      | `Alt + ↑↓`               |
| 現在の行を削除                   | `Shift + DELETE` or `DD` | `Sfhit + DELETE` or `DD` |
| あとからクォーテーションをつける | `Ctrl + Shift + '`       | `Shift + 2`              |
| コードの再フォーマット           | `Ctrl + Alt + L`         | `Shift + Alt + F`        |
| 外部ドキュメント                 | `Ctrl + F1`              | `Ctrl + F1`              |

**ファイル関連**

| 機能             | IntelliJ     | VSCode     |
| :--------------- | :----------- | :--------- |
| ファイルの保存   | `Ctrl + S`   | `Ctrl + S` |
| ファイル名を変更 | `Shift + F6` | `F2`       |
| ファイルの削除   | `DELETE`     | ` DELETE`  |

**実装関連**

| 機能                                   | IntelliJ                 | VSCode                   |
| :------------------------------------- | :----------------------- | :----------------------- |
| クラスへ移動                           | `Ctrl + N`               | `Ctrl + P`               |
| ファイルへ移動                         | `Ctrl + Shift + N`       | `Ctrl + P`               |
| シンボルへ移動                         | `Ctrl + Alt + Shift + N` | `Ctrl + T`               |
| 次の/前のエディタタブに移動            | `Alt + Right/Left`       | `Ctrl + Tab`             |
| 前のツールウィンドウに戻る             | `F12`                    | `Ctrl + Shift + Tab`     |
| 行へ移動                               | `Ctrl + G`               | `Ctrl + G`               |
| 宣言に移動                             | `Ctrl + B`               | `F12`                    |
| 実装に移動                             | `Ctrl + Alt + B`         | `Ctrl + F12`             |
| クイック定義ルックアップを開く         | `Ctrl + Shift + I`       | `Ctrl + Shift + I`       |
| 型宣言に移動する                       | `Ctrl + Shift + B`       | `Ctrl + Shift + B`       |
| スーパーメソッド/スーパークラスへ移動  | `Ctrl + U`               | `Ctrl + U`               |
| 前のメソッド/次のメソッドへ移動        | `Ctrl + Up/Down`         | `Ctrl + Up/Down`         |
| コードブロックの終端/始端に移動する    | `Ctrl + Shift + Up/Down` | `Ctrl + Shift + Up/Down` |
| ファイル構造ポップアップ               | `Ctrl + F12`             | `Ctrl + Shift + O`       |
| タイプ階層                             | `Ctrl + H`               | `Ctrl + H`               |
| メソッド階層                           | `Ctrl + Alt + H`         | `Ctrl + Shift + H`       |
| ソース編集/ソース表示                  | `Ctrl + Shift + F4`      | `Ctrl + Shift + F4`      |
| ナビゲーションバーの表示               | `Ctrl + Shift + F1`      | `Ctrl + Shift + F1`      |
| ブックマーク切り替え                   | `Ctrl + F11`             | `Ctrl + F9`              |
| ブックマーク（ニーモニック）の切り替え | `Ctrl + Shift + F11`     | `Ctrl + Shift + F9`      |

## 文字編集関連

### コメントアウト

| IntelliJ   | VSCode     |
| :--------- | :--------- |
| `Ctrl + /` | `Ctrl + /` |

**IntelliJ**
`Ctrl + /` で、コメントアウトできます。

![qiita1.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/bd5c81e2-76f7-0fd8-4a33-70cf29f64d53.gif)

**VSCode**
`Ctrl + /`　で、コメントアウトできます。

![qiita2.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3fcb80d9-b3a3-f4f1-39e9-f5d69e152c01.gif)

### 同じ変数を一括更新

| IntelliJ                 | VSCode |
| :----------------------- | :----- |
| `Ctrl + Alt + V` or `F2` | `F2`   |

**IntelliJ**

`Ctrl + Alt + V` or `F2`　で、同じ変数を一括更新できます。

`Ctrl + Alt + V` と `F2` があります。(`F2`は自分で設定したかもしれません)
`Ctrl + Alt + V` と `F2`　の大きな違いは、`Ctrl + Alt + V`はその箇所だけを選択するかというワンアクションを余計に踏まないといけないことです。`F2`を使うといりません。

![qiita3.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2bc3d759-c1f1-2211-6162-d960d0c0bed2.gif)

**VSCode**

`F2`　で、同じ変数を一括更新できます。

![qiita5.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/253b1fda-e51a-9efc-2758-03722ced283b.gif)

### 行の入れ替え

| IntelliJ                                 | VSCode                 |
| :--------------------------------------- | :--------------------- |
| `Ctrl + Shift + ↑` or `Ctrl + Shift + ↓` | `Alt + ↑` or `Alt + ↓` |

**IntelliJ**

`Ctrl + Shift + ↑` or `Ctrl + Shift + ↓`　で、行の入れ替えができます。

![qiita6.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3da151a3-9b67-0c5a-7bc5-96602eee2367.gif)

**VSCode**

`Alt + ↑` or `Alt + ↓`　で、行の入れ替えができます。

![qiita7.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/0d1dd73c-49fc-ca69-8900-39381f3f8ce3.gif)

### 現在の行を削除

| IntelliJ                 | VSCode                   |
| :----------------------- | :----------------------- |
| `Shift + DELETE` or `DD` | `Sfhit + DELETE` or `DD` |

自分は、行削除＋クリップボードにコピーすることが多いので、`DD`を使っています。

**IntelliJ**

`Shift + DELETE`　で、現在の行を削除できます。
![qiita8.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/cc2b747c-0b29-889b-4f79-c233ae3e6688.gif)

**VSCode**

`Shift + DELETE` で、現在の行を削除できます。
![qiita9.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/79f31633-f66b-1c52-95a4-b3a8c1a49b0b.gif)

### "あとからクォーテーションをつける"

| IntelliJ           | VSCode      |
| :----------------- | :---------- |
| `Ctrl + Shift + '` | `Shift + 2` |

**IntelliJ**

`Ctrl + Shift + '` で、"あとからクォーテーションをつける"ができます。
![qiita11.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/f1ab284f-d7f9-93c6-c865-ebccc36be987.gif)

**VSCode**

`Shift + 2` で、"あとからクォーテーションをつける"ができます。

![qiita12.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/b35784aa-5652-036f-c1bf-2464b355adf8.gif)

## ファイル編集関連

### ファイル名を変更

| IntelliJ             | VSCode |
| :------------------- | :----- |
| `Shift + F6` or `F2` | `F2`   |

`F2`は自分で設定したかもしれません。

**IntelliJ**

`Shift + F6` or `F2` で、ファイル名を変更できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8d1a8692-e636-e742-ec67-bdb4376b4651.png)

**VSCode**

`F2` で、ファイル名を変更できます。
![qiita13.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2ec35411-f177-e47a-41fc-fd5b7367439c.gif)

### ファイルを削除

| IntelliJ | VSCode   |
| :------- | :------- |
| `DELETE` | `DELETE` |

##　検索周り

### search everywhere

| IntelliJ        | VSCode     |
| :-------------- | :--------- |
| `Shift + Shift` | `Ctrl + P` |

**IntelliJ**

`Shift + Shift` で、検索ウィンドウが開きます。

![qiita14.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/c3b21ca6-b301-c23d-baf4-b987ef0e258d.gif)

**VSCode**

`Ctrl + P` で、検索ウィンドウが開きます。

![qiita15.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/b00de79f-8c56-c1a9-9089-65bae4b6f73d.gif)

## デバッグ、コンパイル関連

| 機能                                           | IntelliJ    | VSCode      |
| :--------------------------------------------- | :---------- | :---------- |
| ビルド                                         | `Ctrl + F9` | `Ctrl + F5` |
| 選択したファイル、パッケージのコンパイルと実行 | `Ctrl + F9` | `Ctrl + F5` |
| デバッグ                                       | `F9`        | `F5`        |

## 実装関連

| 機能                                   | IntelliJ                 | VSCode                   |
| :------------------------------------- | :----------------------- | :----------------------- |
| クラスへ移動                           | `Ctrl + N`               | `Ctrl + P`               |
| ファイルへ移動                         | `Ctrl + Shift + N`       | `Ctrl + P`               |
| シンボルへ移動                         | `Ctrl + Alt + Shift + N` | `Ctrl + T`               |
| 次の/前のエディタタブに移動            | `Alt + Right/Left`       | `Ctrl + Tab`             |
| 前のツールウィンドウに戻る             | `F12`                    | `Ctrl + Shift + Tab`     |
| 行へ移動                               | `Ctrl + G`               | `Ctrl + G`               |
| 宣言に移動                             | `Ctrl + B`               | `F12`                    |
| 実装に移動                             | `Ctrl + Alt + B`         | `Ctrl + F12`             |
| クイック定義ルックアップを開く         | `Ctrl + Shift + I`       | `Ctrl + Shift + I`       |
| 型宣言に移動する                       | `Ctrl + Shift + B`       | `Ctrl + Shift + B`       |
| スーパーメソッド/スーパークラスへ移動  | `Ctrl + U`               | `Ctrl + U`               |
| 前のメソッド/次のメソッドへ移動        | `Ctrl + Up/Down`         | `Ctrl + Up/Down`         |
| コードブロックの終端/始端に移動する    | `Ctrl + Shift + Up/Down` | `Ctrl + Shift + Up/Down` |
| ファイル構造ポップアップ               | `Ctrl + F12`             | `Ctrl + Shift + O`       |
| タイプ階層                             | `Ctrl + H`               | `Ctrl + H`               |
| メソッド階層                           | `Ctrl + Alt + H`         | `Ctrl + Shift + H`       |
| ソース編集/ソース表示                  | `Ctrl + Shift + F4`      | `Ctrl + Shift + F4`      |
| ナビゲーションバーの表示               | `Ctrl + Shift + F1`      | `Ctrl + Shift + F1`      |
| ブックマーク切り替え                   | `Ctrl + F11`             | `Ctrl + F9`              |
| ブックマーク（ニーモニック）の切り替え | `Ctrl + Shift + F11`     | `Ctrl + Shift + F9`      |
| 番号付きブックマークへ移動             | `Ctrl + 0`               | `Ctrl + 0`               |

(力尽きたので、時間があるときに追記します。)

## 雑談

VSCode で、Screencast Mode を使うと、キーボードの入力を表示できます。
コマンドパレット(Ctrl + Shift + P)から、`Screencast Mode` を有効にすると、キーボードの入力が表示されます。
IntelliJ にはこの機能ないのかな...？

## 最後に

IntelliJ を本格的に業務で使いだして、半年が経ちました。
最初は、IntelliJ 使いづらい...となっていましたが、今では、IntelliJ の方が速く実装できる場面が多くなりました。
特に Go 言語や Python, 大きめの C++のプロジェクトでは、IntelliJ の方が速く実装できることが多いです。
宗教上の理由で、JAVA, Ruby は絶対にコードを書かないぞ！という強い意志がありますので、IntelliJ(JAVA) や WebStorm は使いませんが、Goland, Clion, PyCharm は、今後も使っていきたいと思います。
