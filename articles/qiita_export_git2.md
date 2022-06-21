---
title: git基礎（実践）
tags: Git 勉強会
author: ekusiadadus
slide: false
---

# git 基礎（実践）

環境:Windows10,WSL Ubuntu 18.04, Git2.28.0, (Backlog git)
言語:C++(好きな言語使ってください)
実際に、[理論編の話](https://qiita.com/drafts/48d6a625a9e0be2852d2)を、1 から git で開発していきます。

## §1 git リポジトリを作成する

### 1.ディレクトリを作成する

git リポジトリを作成するには、ディレクトリが必要です。/home/[user]/以下に"demo"ディレクトリを作成します。
Ubuntu では、mkdir でディレクトリを作成できます。
![eac69da3-7747-4274-a8d3-2dc5803214d7.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/467a559f-b085-04f8-51e5-0f555171a90a.gif)

git init コマンドが成功すると、.git ディレクトリ(隠し)が作成されます。

```
ls -la .git
```

として、.git のフォルダ構成をみると以下のようになっています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/14d82dc4-5af7-ec45-0410-61f8e19eb80b.png)

オブジェクトと参照の存在が確認できます。(objects と refs)
この 2 つのディレクトリに、全てのリポジトリデータがこの 2 つのディレクトリの中に格納されています。

```
git hlep init
```

と打ち込んでみましょう。そうすると、init コマンドの詳細が見ることができます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3534d17a-71ec-bd8c-24e1-a8026d3ef4b3.png)

```
git status
```

と打ち込んでみましょう。そうすると、今何が起こっているのかを確認することができます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/01dfe6b4-3bfe-4b09-3f36-803db1043f87.png)

現状は、なにも操作を行っていないので、"No commits yet"となっています。
git 基礎(理論)で、そのコマンドがどのような操作(DAG である git の履歴に対する処理)を行っているのかを考えてみましょうと、書きましたが、現状は DAG がただ始点のみで、辺が張られていないので、No commits yet となっています。

### 2.ファイルを追加する

git リポジトリにファイルを追加してみましょう！
"hello world"という文字列内容の書かれた、hello.txt を作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/b58b6e11-1e31-8a09-ef32-13ec8f857eb8.png)

```
demo\<root directry>
|
+-hello.txt(blob, contents="hello world")
```

さて、これを新しいスナップショットとして、このディレクトリの一番最初の状態として、変更履歴を保存します。

```
git snapshot
```

git には上のコマンドは存在しません。
現在のディレクトリ全てをスナップショットとして保存するようなコマンドはありません。理由はいくつかありますが、git は、次にどのような変更をスナップショットに加えるかというオプションを与えて、より柔軟な管理方法を提供するためです。
git は、ステージングという概念を持っています。git は、次のスナップショットにどのような変更を加えるかというオプションを与えます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/bf5aeea9-f328-c234-1b35-03fc61fc2a65.png)

```
git status
```

上から、git は、リポジトリに変更があったことを認識していますが、それをスナップショットに加えることはしていません。
なぜなら、これは、ユーザにこの変更を次のスナップショットに加えるか、否かのオプションを与えるためです。

```
git add hello.txt
```

と打つと、
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/39de26ee-de59-db9e-b615-1691a7ef1c96.png)

変更が追加されました。これは、スナップショット(git commit)にこの変更を加えるという意味です。

```
git commit
```

を打ってみましょう。
![888f18ce-882e-4860-9b2f-61c3adc1f58d.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/973046df-9146-fa3b-0542-2c05b20496cb.gif)

デフォルトエディタが立ち上がり、コミットメッセージを入力します。コミットメッセージは、あとからなぜその変更を加えたのかが一目でわかるようなメッセージであるべきです。しかし、今回はテストなので"Add hello.txt"としています。
よいコミットメッセージの書き方を載せておきます。OSS などで、チーム開発するときは、意味のあるメッセージを残すように心がけましょう！
[Write good commit messages!](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
[Even more reasons to write good commit messages!](https://chris.beams.io/posts/git-commit/)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/97c00e20-ce9c-032c-1cbc-0ccf413a7a83.png)

commit に成功すると、上のようになります。
ここで、[master(root-commit)f75edd5]の f75edd5 は今の commit のハッシュ値です。ここでの commit は正確には、tree のポインタを保持しています。

```
git cat-file -p f75edd5
```

と打つと、
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/d934fd70-0b34-9a91-371f-afa96168cac1.png)

tree のハッシュ値が保持されています・・・。
更に、

```
git cat-file -p 68aba62e560c0ebc3396e8ae9335232cd93a3f60
git cat-file -p 3b18e512dba79e4c8300dd08aeb37f8e728b8dad
```

と打つと、上のように hello.txt の変更内容がわかります。
つまり、"git commit"は、参照の集合をスナップショットとして保存していることがわかります。

### 3.ログを確認する

git の log コマンドは、変更履歴を視覚的にわかりやすく表示してくれます。

```
git log
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3dda737d-15f2-dbf6-cb80-5f45a26b10d7.png)

上のように、git log は変更履歴を表示してくれます。現状は、一つの commit しかないので、一つの変更履歴のみ表示されます。
新しく commit してみましょう。

```
echo "another line" >> hello.tx
git add hello.txt
git commit -m "Add another line in hello.txt
git log
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/66145620-bf87-62a9-9cbf-e9d08870baaa.png)

現状は、2 つのコミットしかないですが、git log になんのオプションも書かないと線形的に書かれます。

```
git log --all --graph --decorate
```

のようにオプションを付けることでより視覚的にわかりやすくログを表示してくれます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2d968854-a1eb-778c-e546-3cc443695667.png)

左辺の赤い線がグラフの辺を表しています

### 4.オブジェクトと参照

次は、
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2d96a599-fb8f-997a-81c7-99ad2733abfd.png)

これを見ていきましょう。
git 基礎(理論)でも紹介しましたが、git ではハッシュ関数によって暗号化された参照を人間にとってわかりやすくマッピングします。
master とは、git リポジトリを作成したときに必ず作成される、最新のメインブランチの特別な参照です。今回は、master は、 3800812708f08da07746b7800b880050590ee71e のポインタです。
HEAD も、git リポジトリの特別な参照です。HEAD は、今自分がいる場所を示しています。

### 5.変更履歴を遷移する

HEAD は、今どこにいるかを示す参照でした。
いまは、
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/d7a8c567-a05f-a814-c66e-47dedde965a6.png)

```
 commit 3800812708f08da07746b7800b880050590ee71e (HEAD -> master)
```

ここにいます。

```
git checkout f75edd575792baddf801cf3f7e1479f1b9a6da14
```

とすると変更履歴を遡ります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/a60ae030-cafe-415a-b299-3061fea270bc.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/92554bc9-348b-b01f-7f3e-6402234386b9.png)

この状態で、"cat hello.txt"とすると、hello.txt に"another line"を加える前の状態に戻ります。
また、HEAD(今いる場所)が、ひとつ前のスナップショットに戻っていることに注目してください!!

```
git checkout master
```

とすると、最新のスナップショットの場所に戻ります。(master は、最新のスナップショットの参照)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/fcc57974-c10a-7c9c-cc0e-9a4bc322a0df.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/945a6797-803f-b4b4-b4c7-f73628c7bd77.png)

hello.txt も、最新の状態になっていることが確認できます。
git add コマンドを行わずに、git checkout を行うと変更履歴が保存されていない状態で、状態遷移をすると、blob や tree が書き換わってしまうので、注意してください。

### 6.diff コマンド

最新のスナップショットの状態で、hello.txt の内容を書き換えてみましょう
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/6122c6be-29a7-f1ae-d48e-c7c68a0be36e.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2ff09b74-6951-a6f0-b35d-751151111583.png)

git diff コマンドでは、現在のスナップショット(HEAD)から今の状態で、変更された内容を出力します。
オプションとして、

```
git diff ポインタ1 ポインタ2 ファイル名
```

でポインタ 1 の示すスナップショットと、ポインタ 2 の示すスナップショットのファイルの変更された内容を出力します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/d232bfbb-a821-5b3e-828f-b3a073c4c5d5.png)

### 7.ブランチ

"Hello" を出力する簡単な C++(好きな言語でいいです)を作成します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/bff68474-a7b8-445e-5229-e5e967e5e41d.png)

そうして、新しく、"cat"ブランチを作成します。

```
git checkout -b cat
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3c74ce27-e9f2-5ef9-9224-930a2c5faf98.png)

新しいブランチ"cat"が作成されたのが見えます。
HEAD がさしている master が現在のブランチで、新しいスナップショットを作成したときは、master に変更履歴が加えられます。

```
git checkout cat
```

とすると、ブランチが cat になります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/fcf056ce-7261-a30c-689d-5942362205bc.png)

animal.cpp をコマンドライン引数で、"cat"を持つとき、"Meow!"と標準出力するように書き換えます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/82464934-90e3-ac90-1e1b-e22775e6265a.png)

git にスナップショットを保存します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/50921a13-cc19-cf17-8e88-9f0a5ccf84b1.png)

cat ブランチに cat 関数を加えたので、master ブランチとは違う状態にいます。
このような状態を,parallel programming と言ったりします。
同様にして、dog ブランチを追加して、コマンドライン引数に dog を持つとき、"Woof!"と出力するように書き換えます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/35b8c146-810e-1af6-f285-ddbb203c18f9.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/687d8caf-e684-2433-06db-cbd6ce007246.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/d49ef5b4-4bd7-6d53-faa2-72f099127bd3.png)

上のログを見るとグラフ構造がよくわかると思います。これも git log オプションのおかげです。
図を書くと

```
〇<---〇<---〇(master)＋〇cat
 　　　　　　　｜
　　　　　　　＋〇dog(HEAD)
```

のように master から、dog,cat が分裂しています。

これから、master に dog,cat ブランチを統合していきます。
まず、HEAD を master にします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/7ad8f38c-2c4b-c55d-3aa6-cad63ecdaa71.png)

その後、

```
git merge cat
```

で cat ブランチをマージします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/d7d451e1-c649-8576-be12-d369f6c4c170.png)
この状態で、animal.cpp は cat ブランチの animal.cpp となっています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/b0c9e9aa-5639-14b5-9bf4-bf71add7895b.png)
次に、dog ブランチをマージします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2ef106e9-731f-f421-517a-63fc641782f9.png)

CONFLICT が出ました。これは、二つの平行するブランチのマージの衝突が起こったことを意味します。Auto-merging が行われていますが、この衝突が起こった時はこの課題を修正することが必要です。このような、マージの衝突が起こった場合に使うツールが git に入っています。それが、git mergetool です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8e8370f7-eda9-7572-c7b7-e9564ad84247.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/9ff1b7de-ed79-98bc-3e2a-20fbebc84aa4.png)

このツールを使うと、上のようにマージの衝突の詳細を比較してくれます。
このツールを使って、修正を施していきます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/11cc7923-0d0e-9207-b7b7-25e3cb251164.png)

今回は、main 関数内の if 文を else if に書き換える等の修正を施さなければならないです。
そのあと、git に merge を続けてもらいます。

```
git merge --continue
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8c6a26f1-2ca1-b00d-fda1-789b1392efb0.png)

```
〇<---〇<---〇(master)＋〇cat---
 　　　　　　　｜　　　　　　　　　|<---〇master(cat,dogマージ)
　　　　　　　　＋〇dog(HEAD)---
```

マージが成功すると、このようなログになります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/b64b7b49-68ac-436b-a639-a2ae958047eb.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/5ae3831d-1bf7-8980-538a-7cbc9bf869db.png)

(実行するとこんな感じ)
