---
title: 仕事中にワールドカップをばれないように見る、CLIツールをRustで作ってみた
tags: Twitter, WorldCup, CLI, Rust
author: ekusiadadus
slide: false
---

# 仕事中にワールドカップをばれないように見る CLI ツールを Rust で作ってみた

こんにちは、@ekusiadadus です。
日本が死の組をまさかの一位通過して本選出場を決めましたね。

ワールドカップ見たい......けど、仕事中にばれたら怒られる......

そんなときに仕事中にワールドカップを見るための CLI ツールを作ってみました。

(ついさっき PK で負けてしまいました 😢)

## How to use

![image](https://user-images.githubusercontent.com/70436490/205714489-7e4f5874-f6a8-47f9-98d4-011d3930b49c.png)

```
🌸 World Cup 2022 CLI for Japanese football fans 🌸

Usage: samuraicli <COMMAND>

Commands:
  real     ⚽ワールドカップをリアルタイムで確認する
  search   🥅ワールドカップのツイートを取得する
  keisuke  📣本田圭佑の動向を取得する
  help     Print this message or the help of the given subcommand(s)

Options:
  -h, --help  Print help information
```

## 12 月 6 日のクロアチア戦のときに動かした動画

![Ubuntu 22 04 1 LTS 2022-12-06 01-08-36_12](https://user-images.githubusercontent.com/70436490/205720685-f5692fd6-34fa-420a-ae3b-65e4b41c4429.gif)

## 作り方

設計ですが、ツイートを取得する箇所はクリーンアーキテクチャ的に作っています。(結構癖があるかもです)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/3b6d343d-b398-c6c2-076c-019afc13f3ff.png)

CLI 部分とかは、単純に `main.rs` に書いています。
CLI ツールを作るときは、[clap](https://crates.io/crates/clap) を使いました。
文字は[owo-colors](https://crates.io/crates/owo-colors)を使ってランダムで色を付けています。

### 1. Twitter API を使う

Twitter API を使って、ワールドカップのツイートを取得しました。
ここら辺は、[Twitter を BigQuery と JupyterLab で分析してみた ~ Twitter API v2 ~](https://zenn.dev/ekusiadadus/articles/twitter_bigquery_jupyterlab1) で書いたので、そちらを参考にしてください。

### 2. Rust で CLI ツールを作る

CLI ツールを作るときは、[clap](https://crates.io/crates/clap) を使いました。

以下のように書くと、オプションをいい感じに渡せます。

```rust

fn cli() -> Command {
    Command::new("samuraicup")
        .about("🌸 World Cup 2022 CLI for Japanese football fans 🌸")
        .subcommand_required(true)
        .arg_required_else_help(true)
        .allow_external_subcommands(true)
        .subcommand(Command::new("real").about("⚽ワールドカップをリアルタイムで確認する"))
        .subcommand(Command::new("search").about("🥅ワールドカップのツイートを取得する"))
        .subcommand(Command::new("keisuke").about("📣本田圭佑の動向を取得する"))
}

```

ここで受け取った値を、`match` で分岐させて、処理を実行します。

ツイートを受け取ったりする処理を書くときには、tokio を使って非同期処理を書きました。
ここら辺は、クリーンアーキテクチャ的に書いています(と思っています)。

```rust
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    std::env::set_var("RUST_LOG", "info");
    std::env::set_var("RUST_BACKTRACE", "1");
    dotenv().ok();

    let db_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let db_pool_size = std::env::var("DATABASE_POOL_SIZE")
        .ok()
        .and_then(|it| it.parse().ok())
        .unwrap_or(5);
    let bearer_token = std::env::var("BEARER_TOKEN").expect("BEARER_TOKEN not set");

    let app = initializer::new(initializer::Config {
        db_url: db_url,
        db_pool_size: db_pool_size,
        bearer_token: bearer_token,
    })
    .await;

    app.infras
        .ensure_initialized()
        .await
        .expect("Infra initialization error");

    let matches = cli().get_matches();
```

## まとめ

これで仕事中にもばれずにワールドカップを見ることができますね！

...あ、リモートだから関係なかった。
テレビ見よ
イングランド応援します。
