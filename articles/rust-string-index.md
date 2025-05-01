---
title: "なぜ Rust で文字列にインデックスでアクセスできないのか？" # 記事のタイトル
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Rust", "文字列", "Python", "コンパイラ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# なぜ Rust で文字列にインデックスでアクセスできないのか？

Rust の文字列にインデックスでアクセスしようとしたことはありませんか？
しばしば Python や C++のノリで Rust を書いて、めちゃくちゃコンパイラに怒られるということが多々あります。

最初に答えを書いてしまいますが、Rust では文字列にインデックスでアクセスできません。

例えば、次のようなコードを書いたとします。

```Rust
fn main() {
    let s = String::from("hello");
    let c = s[0];
}
```

このコードをコンパイルしようとすると、次のようなエラーが発生します。

```shell
 cargo run
   Compiling rust-string-index v0.1.0 (/home/ekusiadadus/dev/rust-string-index)
error[E0277]: the type `str` cannot be indexed by `{integer}`
 --> src/main.rs:3:15
  |
3 |     let c = s[0];
  |               ^ string indices are ranges of `usize`
  |
  = note: you can use `.chars().nth()` or `.bytes().nth()`
          for more information, see chapter 8 in The Book: <https://doc.rust-lang.org/book/ch08-02-strings.html#indexing-into-strings>
  = help: the trait `SliceIndex<str>` is not implemented for `{integer}`
          but trait `SliceIndex<[_]>` is implemented for `usize`
  = help: for that trait implementation, expected `[_]`, found `str`
  = note: required for `String` to implement `Index<{integer}>`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `rust-string-index` (bin "rust-string-index") due to 1 previous error
```

このエラーは、Rust の文字列が UTF-8 エンコーディングを使用しているため、インデックスでアクセスできないことを示しています。
Rust が文字列に直接インデックスアクセスを許可しない理由は、UTF-8 エンコーディングの仕組みによるものです。Rust の文字列は UTF-8 でエンコードされており、文字（Unicode 文字）のサイズが可変長だからです。

## String と &str の違い

ここで、Rust の文字列型について少し触れておきましょう。Rust には主に 2 つの文字列型があります：

1. `String` - ヒープに割り当てられた、可変長で所有権を持つ文字列
2. `&str` - 文字列スライス。文字列データへの参照（ビュー）

```Rust
fn main() {
    // String型 - 所有権があり、変更可能
    let mut owned_string = String::from("Hello");
    owned_string.push_str(", world!");

    // &str型 - 文字列スライス（参照）
    let string_slice = &owned_string[0..5]; // "Hello"

    // リテラルも&str型
    let literal = "こんにちは"; // 型は&'static str

    println!("{}", owned_string); // "Hello, world!"
    println!("{}", string_slice); // "Hello"
    println!("{}", literal); // "こんにちは"
}
```

どちらの型も内部的には UTF-8 エンコードされたバイト列として保存されており、インデックスでのアクセスに関する制約は同じです。

例えば、次のような文字列を考えてみましょう。

```Rust
fn main() {
    let s = "こんにちは";
    println!("バイト数: {}", s.len()); // 15バイト（各日本語文字は3バイト）
    println!("文字数: {}", s.chars().count()); // 5文字
}
```

「こ」のような日本語文字は 3 バイトを使うため、単純なインデックスアクセスでは意味のある文字単位のアクセスができません。もし s[1]と書いた場合、2 番目の文字（「ん」）ではなく、「こ」の 2 番目のバイトを取得することになります。これは無効な UTF-8 になる可能性があり、Rust はこのような安全でない操作を防止します。

一応、文字列スライスにするとバイトインデックスでアクセスできます。

```Rust
fn main() {
    let s = "こんにちは";

    // バイトインデックスでのスライス
    let slice = &s[0..3]; // "こ" - 最初の文字は3バイト
    println!("{}", slice);

    // 文字の境界に一致しないスライスはパニックを起こす
    // let invalid_slice = &s[0..2]; // パニック: バイト境界が文字の境界と一致しない
}
```

## 文字アクセスの方法とパフォーマンス

文字単位でアクセスしたい場合は、いくつかの方法があります。それぞれ異なるパフォーマンス特性を持っています：

```Rust
fn main() {
    let s = "こんにちは";

    // 方法1: .chars().nth() - O(n)の時間複雑度
    let third_char = s.chars().nth(2).unwrap(); // "に"
    println!("3番目の文字: {}", third_char);

    // 方法2: .chars()からのイテレーション - 全文字に順次アクセスする場合は効率的
    for (i, c) in s.chars().enumerate() {
        println!("文字 {}: {}", i, c);
    }

    // 方法3: バイトレベルでアクセス - 低レベル操作が必要な場合
    for b in s.bytes() {
        println!("バイト: {}", b);
    }

    // 文字単位でのスライス（非効率だが安全）
    let char_slice: String = s.chars().skip(1).take(2).collect();
    println!("文字スライス: {}", char_slice); // "んに"
}
```

`.chars().nth(n)` は内部的にイテレータを進めるため、O(n)の時間複雑度があります。つまり、文字列の長さに比例して処理時間が増加します。これは特に長い文字列で特定のインデックスにアクセスする場合は非効率的です。

一方、`.bytes()` メソッドはバイトレベルのアクセスを提供し、UTF-8 のエンコーディングを直接扱う必要がある低レベルの操作では効率的ですが、文字単位の操作では注意が必要です。

ここで、chars() メソッドを呼び出してから、nth() メソッドを使うことで、指定したインデックスの文字を取得しています。
char 型にすることで、Unicode コードポイントを表すことができ、UTF-8 エンコーディングのバイト数に依存しない形で文字を扱うことができます。
Rust の Char 型は 1 つの Unicode コードポイントを表現します。Rust の char 型は常に 4 バイト（32 ビット）のサイズを持ち、有効な Unicode スカラー値を格納できます。

## 余談

Rust では、文字列にインデックスでアクセスできないのは、UTF-8 エンコーディングの特性によるものですが、他の言語ではこのような制約がない場合があります。
例えば、同様のコードを C++で書くと、次のようになります。

```C++
#include<iostream>
#include<string>
using namespace std;

int main(){

    string s = "hello";
    char c = s[0];

    return 0;
}
```

このコードは、コンパイルエラーは起きず実行もできます。
しかし、C++ では文字列はバイト列として扱われるため、UTF-8 エンコーディングの特性を考慮していません。したがって、インデックスでアクセスすることができますが、UTF-8 の特性を無視しているため、文字列の操作においてバグやセキュリティ上の問題を引き起こす可能性があります。

Python はより柔軟で、文字列は実際には Unicode コードポイントのシーケンスとして扱われます。そのため、マルチバイト文字でもインデックスアクセスが直感的に機能します

```Python
s = "こんにちは"
print(s[0]) # "こ"
print(s[1]) # "ん"
```

Python では、内部的に文字列が Unicode コードポイントとして表現されているため、インデックスアクセスが直感的に動作します。これは Rust の char 型の配列に近い概念です。

### Unicode の複雑さ - 絵文字と結合文字

Unicode 処理の複雑さを示す良い例として、絵文字や結合文字を含む文字列の扱いを見てみましょう。

```Rust
fn main() {
    // 絵文字を含む文字列
    let emoji_string = "Hello 🦀 Rust!";
    println!("バイト数: {}", emoji_string.len()); // 14 (ASCII) + 4 (絵文字) + 6 (ASCII) = 24バイト
    println!("文字数: {}", emoji_string.chars().count()); // 13文字

    // 結合文字を含む文字列
    let combining_string = "e\u{301}"; // é (eと結合アクセント記号)
    println!("バイト数: {}", combining_string.len()); // 3バイト
    println!("文字数: {}", combining_string.chars().count()); // 2文字（見た目は1文字）

    // 家族の絵文字（複数のコードポイントから成る）
    let family_emoji = "👨‍👩‍👧‍👦";
    println!("バイト数: {}", family_emoji.len()); // 25バイト
    println!("文字数: {}", family_emoji.chars().count()); // 7文字（👨,ZWJ,👩,ZWJ,👧,ZWJ,👦）
    println!("グラフェム数: {}", family_emoji.graphemes(true).count()); // unicode-segmentation クレートを使用すると1と数えられる
}
```

上記の例は、Unicode の複雑さを示しています。特に注目すべきなのは家族の絵文字で、これは複数の絵文字とゼロ幅接合子（ZWJ）から構成されていますが、視覚的には単一の絵文字として表示されます。

このような複雑な Unicode 文字列を正確に扱うには、`unicode-segmentation`クレートなどの追加ツールが必要になることがあります。これは、`.chars()`メソッドが Unicode のコードポイント単位で動作し、人間が認識する「文字」（グラフェムクラスタ）単位では動作しないためです。

これらの例は、Rust が単純なインデックスアクセスを許可しない理由をさらに強調しています。文字列の処理は、単純なバイト配列としての取り扱いよりも、はるかに複雑なのです。
