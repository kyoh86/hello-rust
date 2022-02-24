---
title: "gopherのrust学習メモ"
emoji: "⚙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust", "go"]
published: true
---

## これは何

Goをだいぶ長く書いていた私（rustはHello world動かしたくらい）が改めてrustをイチから勉強する上で気になった話題をピックアップして貼っておくメモです。
目下 [The Rust Programming Language 日本語版](https://doc.rust-jp.rs/book-ja/) を読み進めています。

:::message
他のGopherにとっても役に立つかもしれませんが、あまり期待はできないです。
:::

:::message
メモなので時々追記しますが、rustの更新などに応じて見返して訂正するようなことはしません
:::

## hello

```console
$ cargo new hello_rust --bin
$ cd hello_rust
$ edit src/main.rs
```

```rust
fn main() {
  println!("hello, rust!")
}
```

```console
$ cargo run
```

## build

```console
$ cargo build
$ cargo build --release
```

## クレート

パッケージ的なもの。バイナリクレートとライブラリクレートがある。
`Cargo.toml`の`[dependencies]`で依存を記述できる

```toml
[dependencies]

rand = "0.3.14"
```

クレートの取得は `cargo build` 内で行われる。 `Cargo.lock` にバージョンが固定される
`Cargo.toml` の`dependencies`と互換性のある（マイナーバージョンが同じ）バージョン範囲で取得してくれる。

クレートの更新は `cargo update` 。
`Cargo.toml` の`dependencies`と互換性のある（マイナーバージョンが同じ）バージョン範囲で更新してくれる。

## ドキュメント

go docよろしく。

> cargo doc --openコマンドを走らせてローカルに存在する依存すべてのドキュメントをビルドし、ブラウザで閲覧できる機能です。 例えば、randクレートの他の機能に興味があるなら、cargo doc --openコマンドを走らせて、

## エラー処理

エラーが生じるような関数は、`Result` (とその派生型？) を使う。

> このResult型は、列挙型であり、 ~~snip~~ `Result`型に関しては、列挙子は`Ok`か`Err`です。
> `Ok`列挙子は、処理が成功したことを表し、 中に生成された値を保持します。
> `Err` 列挙子は、処理が失敗したことを意味し、`Err`は、処理が失敗した過程や、 理由などの情報を保有します。

> `io::Result`オブジェクトには、呼び出し可能な`expect`メソッドがあります。
> この`io::Result`オブジェクトが`Err`値の場合、`expect`メソッドはプログラムをクラッシュさせ、 引数として渡されたメッセージを表示します。

```rust
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

> もし、`expect` メソッドを呼び出さなかったら、コンパイルは通るものの、警告が出るでしょう:

## エラー処理（match）

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};

println!("You guessed: {}", guess);
```

> `expect`メソッドの呼び出しから`match`式に切り替えることは、
> エラーでクラッシュする動作からエラー処理を行う処理に変更する一般的な手段になります。
> `parse`メソッドは、 `Result`型を返し、`Result`は`Ok`か`Err`の列挙子を取りうる列挙型であることを思い出してください。

> parseメソッドは、文字列から数値への変換に成功したら、結果の数値を保持するOk値を返します。 このOk値は、最初のアームのパターンにマッチし、

> `parse`メソッドは、文字列から数値への変換に失敗したら、エラーに関する情報を多く含む`Err`値を返します。
> この`Err`値は、最初の`match`アームの`Ok(num)`というパターンにはマッチしないものの、 2番目のアームの`Err(_)`というパターンにはマッチするわけです。

## 配列・タプル

https://doc.rust-jp.rs/book-ja/ch03-02-data-types.html

- タプルの "分配"

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

- 配列は配列（固定長）、ベクタはベクタ。

## 文と式

https://doc.rust-jp.rs/book-ja/ch03-03-how-functions-work.html

> Rustは、式指向言語なので、 これは理解しておくべき重要な差異になります。他の言語にこの差異はありませんので、文と式がなんなのかと、 その違いが関数本体にどんな影響を与えるかを見ていきましょう。

> 文とは、なんらかの動作をして値を返さない命令です。
> letキーワードを使用して変数を生成し、値を代入することは文になります。 リスト3-1でlet y = 6;は文です。

```rust
fn main() {
    let y = 6;
}
```

> 式は何かに評価され、~~ snip ~~ 関数呼び出しも式です。マクロ呼び出しも式です。
> 新しいスコープを作る際に使用するブロック({})も式です:

> 以下の式:

```rust
{
    let x = 3;
    x + 1
}
```

> は今回の場合、`4`に評価されるブロックです。その値が、`let`文の一部として`y`に束縛されます。
> 今まで見かけてきた行と異なり、文末にセミコロンがついていない`x + 1`の行に気をつけてください。
> 式は終端にセミコロンを含みません。式の終端にセミコロンを付けたら、文に変えてしまいます。そして、文は値を返しません。
> 次に関数の戻り値や式を見ていく際にこのことを肝に銘じておいてください。

## if

> `if`は式なので、`let`文の右辺に持ってくることができます。リスト3-2のようにですね。

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    // numberの値は、{}です
    println!("The value of number is: {}", number);
}
```

## loop

無限ループ

```rust
loop {
    ...
}
```

条件付きループ

```rust
while number != 0 {
    ...
}
```

for ループ

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        // 値は{}です
        println!("the value is: {}", element);
    }
}
```

> `for`ループと、まだ話していない別のメソッド`rev`を使って範囲を逆順にしたカウントダウンはこうなります:

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

## 所有権

> ヒープデータを管理することが所有権の存在する理由

なるほど。

### 所有権規則

> - Rustの各値は、所有者と呼ばれる変数と対応している。
> - いかなる時も所有者は一つである。
> - 所有者がスコープから外れたら、値は破棄される。

### 変数スコープ

> スコープと変数が有効になる期間の関係は、他の言語に類似しています

💭 実際大差ない。

### より複雑な型

例としてString型。

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str()関数は、リテラルをStringに付け加える

println!("{}", s); // これは`hello, world!`と出力する
```

String型は文字列リテラルとは違う可変のヒープを確保するやつ。
でも、これも `s` が属するスコープが終われば開放される。
所有権の規則通り。

**変数がスコープを抜ける時は `drop` が呼ばれるらしい**

#### 変数とデータの相互作用: ムーブ

整数の再束縛→すべてスタックの上でのことなので、単純に値がコピーされてる。
String型の再束縛→String型が持つ名前lenやcap、確保したヒープ領域へのポインタがコピーされる。

```rust
let s1 = String::from("hoge")
let s2 = s1
```

→ `s1`と`s2`がスコープを抜けた時、`s1`と`s2`が同じヒープを指すポインタを持ってるので、
両方`drop`呼ばれたら二重開放になっちゃう。

→ rustでは、こういうことをすると `s2` に再束縛した時点で、**`s1` は無効化される**

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
println!("{}, world!", s2);
```

確かに。

```
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
```

だから "ムーブ" なんて言い方をするのか。

#### 変数とデータの相互作用: クローン

deep copy的なやつはcloneでヒープごとコピーする。

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

#### 変数とデータの相互作用: コピー

プリミティブな型だとクローンを使わなくても良い。

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

スタックにおさまるから、スタック間のコピーですむ。
逆に言えばクローンしても、結局スタックのコピーなのでやることが変わらない。

こういうスタックに保持される型には `Copy` トレイトというのを配置（？）できるらしい。
型やその一部分でも `Drop` トレイトを実装していると、 `Copy` トレイトで注釈できない。
トレイトの詳細は付録C待ち。

`Copy` の型の例:

> - あらゆる整数型。`u32`など。
> - 論理値型である`bool`。`true`と`false`という値がある。
> - あらゆる浮動小数点型、`f64`など。
> - 文字型である`char`。
> - タプル。ただ、`Copy`の型だけを含む場合。例えば、`(i32, i32)`は`Copy`だが、 `(i32, String)`は違う。

## 所有権と関数

関数に渡した時点で引数へのムーブが起きるので、関数が終わると `drop` される。

```rust
fn main() {
    let s = String::from("hello");  // sがスコープに入る

    takes_ownership(s);             // sの値が関数にムーブされ...
                                    // ... ここではもう有効ではない

    let x = 5;                      // xがスコープに入る

    makes_copy(x);                  // xも関数にムーブされるが、
                                    // i32はCopyなので、この後にxを使っても
                                    // 大丈夫

} // ここでxがスコープを抜け、sもスコープを抜ける。ただし、sの値はムーブされているので、何も特別なことは起こらない。
//

fn takes_ownership(some_string: String) { // some_stringがスコープに入る。
    println!("{}", some_string);
} // ここでsome_stringがスコープを抜け、`drop`が呼ばれる。後ろ盾してたメモリが解放される。
  // 

fn makes_copy(some_integer: i32) { // some_integerがスコープに入る
    println!("{}", some_integer);
} // ここでsome_integerがスコープを抜ける。何も特別なことはない。
```

> takes_ownershipの呼び出し後にsを呼び出そうとすると、コンパイラは、コンパイルエラーを投げるでしょう

明確でいいなコレ。

### 戻り値とスコープ

戻り値は、戻した値を受け取る側にムーブする。
💭 いわゆるC言語的なalloc/freeとはここが違うポイントだな。スコープに束縛されるのではなく、値の所有権に束縛される。returnは所有権をムーブさせるってことか。凄い。凄いぞ！

受け取った変数をそのままreturnすれば、呼び出し元で束縛し直すことで改めて所有権を得ることもできる。

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    //'{}'の長さは、{}です
    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len()メソッドは、Stringの長さを返します

    (s, length)
}
```

とはいえ、全部所有権を維持するためにコレをやるのは大げさなので、ここで **参照** を持ち出すらしい。ほう。

## 続き

https://doc.rust-jp.rs/book-ja/ch04-02-references-and-borrowing.html

ここから読む。
