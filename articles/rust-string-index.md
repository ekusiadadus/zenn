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
 cargo run
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

文字単位でアクセスしたい場合は、.chars()メソッドを使用して文字イテレータを取得し、それを操作するのが安全です：

```Rust
fn main() {
let s = "こんにちは";

    // 3番目の文字にアクセス（0からカウント）
    let third_char = s.chars().nth(2).unwrap(); // "に"
    println!("3番目の文字: {}", third_char);

    // 文字単位でのスライス（非効率だが安全）
    let char_slice: String = s.chars().skip(1).take(2).collect();
    println!("文字スライス: {}", char_slice); // "んに"

}
```

ここで、chars() メソッドを呼び出してから、nth() メソッドを使うことで、指定したインデックスの文字を取得しています。
char 型にすることで、Unicode コードポイントを表すことができ、UTF-8 エンコーディングのバイト数に依存しない形で文字を扱うことができます。
Rust の Char 型は 1 つの Unicode コードポイントを表現します。Rust の char 型は常に 4 バイト（32 ビット）のサイズを持ち、有効な Unicode スカラー値を格納できます。x

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
