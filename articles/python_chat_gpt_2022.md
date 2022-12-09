# 今話題の ChatGPT を Python でいい感じに使ってみよう！

すでにご存知の方も多いと思いますが、ChatGPT は一世を風靡している強力な自然言語処理（NLP）ツールです。人間のようなテキストの生成から、チャットボットやバーチャルアシスタントの構築まで、幅広い用途で利用されています。

ChatGPT の人気の理由の 1 つは、大量のテキストデータで学習させた強力なオープンソースの GPT-3 言語モデルの上に構築されていることです。これにより、ChatGPT は非常に現実的で一貫性のあるテキストを生成することができ、NLP に携わる人にとって貴重なツールとなっています。

しかし、ChatGPT が本当に特別なのは、世界で最も人気のあるプログラミング言語の 1 つである Python で使用するように設計されていることです。Python の基本的な知識があれば、誰でも簡単に ChatGPT をプロジェクトに組み込んで、その強力な機能を使い始めることができるのです。

この記事では、ChatGPT の詳細と、Python でどのように使用できるかを見ていきます。また、ChatGPT の潜在的なアプリケーションのいくつかと、それがどのように良い方向に使われるのかについて説明します。

(ここまで ChatGPT 君)

ChatGPT すごいですよね...
GitHub 見ると最近の Star 数が多くついているのは ChatGPT 関連です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/8dae9c65-feac-af88-a8c2-cacaff0887a6.png)

ということで ChatGPT の Python パッケージ [PyChatGpt](https://github.com/rawandahmad698/PyChatGPT) を触ってみました！

https://github.com/rawandahmad698/PyChatGPT

## インストール

```
pip install chatgptpy --upgrade
```

いろんなパッケージがあるのですが、一番使いやすいと思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/0c1e6960-8a1b-cf0a-c036-01912b074bfe.png)

## 使ってみる

インストール成功したら、`chat.py` ファイルを作って実際に使ってみましょう。

```python:chat.py
from pychatgpt import Chat, Options

options = Options()

options.track = True

options.chat_log = "chat_log.txt"
options.id_log = "id_log.txt"

chat = Chat(email="email", password="password", options=options)
chat.cli_chat()
```

`email` と `password` は [OpenAI](https://beta.openai.com/login/) のアカウントを作成して、そのアカウントのメールアドレスとパスワードを入れてください。

```
python3 chat.py
```

だけで疑似 ChatGPT CLI ツールが体験できます！

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/de27586c-8955-cd52-3b0a-efd8ad6f9bfc.png)

```
You: おはよ！
Chat GPT: こんにちは！ 私は Assistant です。どういたしまして？
You: Qiitaのアドベントカレンダー書かないといけなくて憂鬱なんだよね...
Chat GPT: 「Qiita のアドベントカレンダー」とは何でしょうか？ 私は知りません。ご存知の方がいらっしゃいましたら、是非ご教授ください。


You: [A[A[B
Chat GPT: [Status Code] 403 | [Response Text] <!DOCTYPE html PUBLIC '-//W3C//DTD XHTML 1.0 Transitional//EN' 'http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd'><html xmlns='http://www.w3.org/1999/xhtml'><head><meta content='text/html; charset=utf-8' http-equiv='content-type'/><style type='text/css'>body { font-family:Arial; margin-left:40px; }img  { border:0 none; }#content { margin-left: auto; margin-right: auto }#message h2 { font-size: 20px; font-weight: normal; color: #000000; margin: 34px 0px 0px 0px }#message p  { font-size: 13px; color: #000000; margin: 7px 0px 0px 0px }#errorref { font-size: 11px; color: #737373; margin-top: 41px }</style><title>Microsoft</title></head><body><div id='content'><div id='message'><h2>The request is blocked.</h2></div><div id='errorref'><span>0ZlSTYwAAAAAVbBAdZOMlRrC59NnC25S0VFlCRURHRTA3MTAAZTY2YjhiMDMtMDc5My00NDA5LTk3NzMtMmU2MTJlNzFhMWUz</span></div></div></body></html>
You: 12月にIT関連の記事を書くイベントのことなんだけど、時間があんまりなくてさ...
Chat GPT: それは大変そうですね。12月にIT関連の記事を書くイベントは、時間が限られているようです。そのような状況では、まずは何を書くかを明確にすることが重要です。そして、時間を効率よく使うために、記事の執筆にかかる時間を正確に見積もることも大切です。また、他の参加者やイベントのオーガナイザーと協力し合い、タスクを分担することも検討してみると良いでしょう。
You: 励まして！
Chat GPT: いえいえ、それは大丈夫です。そういう時には、まずは自分自身に自信を持つことが大切です。自分ができることを確認し、自分にとって可能な範囲で最善を尽くすことが大切です。そして、自分が書く記事がどのようなものであるかを心に留めておくことも大切です。そうすれば、書くことを楽しむことができるでしょう。最後に、自分の努力が報われるよう祈っています。頑張ってください！

```

```
You: ChatGPT を Python で使ってみる記事を書いて！
Chat GPT:
[省略]
今回は、ChatGPTの詳細と、Pythonでどのように使用できるかを見ていきます。
また、ChatGPTの潜在的なアプリケーションと、それがどのように良い方向に使われるのかについて説明します。

まず、ChatGPTがどのように機能するかについて説明します。ChatGPTは機械学習モデルであり、与えられた入力に基づいてテキストを生成するように訓練されています。
この入力は、単一の単語、文章、または段落全体であることができ、ChatGPTは入力に関連する応答を生成します。

ChatGPTの大きな特徴の一つは、人間のようなテキストを生成する能力です。
これは、膨大なテキストデータで学習させたGPT-3言語モデルを用いているためです。
これにより、ChatGPTは適切な応答だけでなく、非常に首尾一貫した自然な響きを持つ応答を生成することができます。

PythonでChatGPTを使用することは非常に簡単です。
Python Package Index (PyPI)で公開されているChatGPT Pythonパッケージをインストールすればよいだけです。
パッケージのインストールが完了したら、ChatGPTクラスをインポートし、クラスのインスタンスを作成することで、PythonコードでChatGPTを使用することができます。

以下は、PythonでChatGPTを使用する例です。
```

うん。めちゃくちゃにすごい。

だけど、ブラウザじゃないから最後のサンプルコードが見れないのが残念。
他にも面白いツールがあったので、ご紹介します。

## auto-commit

https://github.com/m1guelpf/auto-commit

[auto-commit](https://github.com/m1guelpf/auto-commit) は、コードの差分を読み取って自動的にコミットメッセージを書いてくれる CLI ツールです。
