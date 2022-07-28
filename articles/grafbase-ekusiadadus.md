---
title: "grafbase で GraphQL やってみたけど動かない"
emoji: "🍉" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["GraphQL"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# grafbase で GraphQL やってみたけど動かない

## grafbase で、プロジェクトを作成する

![](/images/grafbase/1.png)

上のようなUIで、かっこいい。

![](/images/grafbase/2.png)

テンプレートは、いくつか存在している様子。

todo用のスキーマ

```graphql
type TodoList @model {
  id: ID!
  title: String!
  todos: [Todo]
}

type Todo @model {
  id: ID!
  list: TodoList
  title: String!
  complete: Boolean!
}

```

ブログ用のスキーマ

```graphql
type User @model {
  id: ID!
  email: String!
  name: String!
  posts: [Post]
}

type Post @model {
  id: ID!
  title: String!
  content: String!
  user: User!
  comments: [Comment]
}

type Comment @model {
  id: ID!
  content: String!
  user: User!
  post: Post!
}
```

![](/images/grafbase/3.png)

こんな感じで、GitHubにリポジトリも作成してくれる。

ここで、問題発生。

普通にデプロイエラーになる。

https://twitter.com/ekusiadadus/status/1552797936588922881?s=20&t=Jk6Wvqx2pzrSEod0y1ADIg

![](/images/grafbase/4.png)

GitHubで、適当にコミットをプッシュすると正常判定されるらしい...(なんで)

![](/images/grafbase/5.png)


playgroundは、ずーとロード中でうごかない:(

```
curl \
-X POST \
-H "content-type: application/json" \
-H "authorization: Bearer [APIキー]" \
-d '{"query":"{ __schema { types { name } } }"}' \
"https://blog-grafbase-main-ekusiadadus.grafbase.app/graphql"
```

こんな感じで、投げられるみたい...

動くまで、しばらく待機