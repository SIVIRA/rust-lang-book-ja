## 関数

関数はRustのコードに広がっています。すでに言語の中で最も重要な機能の1つ、すなわち多くのプログラムの入り口である `main`関数を見てきました。新しい関数を宣言できる `fn`キーワードも見たことがあります。

Rustコードは、関数名と変数名の従来のスタイルとして*snake case*を使用します。 snakeの場合、すべての文字は小文字で、単語は別々になっています。これは関数定義の例を含むプログラムです。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

Rustの関数定義は `fn`で始まり、関数名の後に括弧が付きます。中括弧は、関数本体の始まりと終わりをコンパイラに伝えます。

定義した関数は、その名前の後ろに括弧を付けて呼び出すことができます。`another_function`はプログラムで定義されているので、`main`関数の中から呼び出すことができます。ソースコードの`main`関数の*後*に`another_function`*を定義しました。それ以前にも定義できます。Rustはどこで関数を定義するかは気にせず、どこかで定義されているだけです。

関数をさらに探索するために*functions*という名前の新しいバイナリプロジェクトを開始しましょう。`another_function`の例を*src/main.rs*に置き、それを実行してください。次の出力が表示されます。

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28 secs
     Running `target/debug/functions`
Hello, world!
Another function.
```

これらの行は、`main`関数に現れる順に実行されます。まず、"Hello、world！"メッセージが表示され、次にanother_functionが呼び出され、そのメッセージが表示されます。

### 関数のパラメータ

関数はシグネチャの一部である特殊変数である*パラメータ*を持つように定義することもできます。関数にパラメータがある場合は、それらのパラメータの具体的な値を与えることができます。技術的には具体的な値は*引数*と呼ばれますが、カジュアルな会話では、関数の定義内の変数または関数を呼び出すときに渡される具体的な値に対して、*パラメータ*と*引数*と呼びます。

次の例は`another_function`をパラメータを使うように書き直しました。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

このプログラムを実行してみてください。 次の出力が得られるはずです。

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21 secs
     Running `target/debug/functions`
The value of x is: 5
```

`another_function`の宣言には、`x`という名前のパラメータが1つあります。`x`の型は`i32`として指定されます。`5`が`another_function`に渡されたとき、`println!`マクロは中括弧のペアが書式文字列にあったところに`5`を置きます。

関数のシグネチャでは、各パラメータの型を宣言する必要があります。これは、Rustの設計における意図的な決定です。関数の定義に型の注釈が必要なことは、コンパイラがコードのどこかでそれらを使用する必要がほとんどないことを意味します。

関数に複数のパラメータを指定する場合は、パラメータ宣言を次のようにカンマで区切ります。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    another_function(5, 6);
}

fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

この例では、2つのパラメータを持つ関数を作成します。どちらも`i32`型です。この関数は、両方のパラメータに値を出力します。関数のパラメータはすべて同じ型である必要はないことに注意してください。

このコードを実行してみましょう。 あなたの*functions*プロジェクトの*src/main.rs*ファイルにあるプログラムを上記の例に置き換えて、`cargo run`を使って実行してください。

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/functions`
The value of x is: 5
The value of y is: 6
```

`x`の値として`5`, `y`の値として`6`が渡されたので、二つの文字列はこれらの値で出力されます。

### 関数本体

関数本体はオプションで式で終わる一連のステートメントで構成されます。これまでは、式を終了することなく関数のみを扱ってきましたが、式の一部として式を見てきました。Rustは式ベースの言語なのでこれは重要な違いです。他の言語でも同じ区別がないので、どのステートメントや式がどのようなものか、その違いが関数の本体にどのように影響するかを見てみましょう。

### 文と式

実際にはすでにステートメントと式を使用しています。 *文*は何らかのアクションを実行し、値を返さない命令です。*式*は結果の値に評価されます。いくつかの例を見てみましょう。

変数を作成し`let`キーワードを使って変数に値を代入するのはステートメントです。リスト3-1では、`let y = 6;`はステートメントです：

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let y = 6;
}
```

<span class="caption">リスト 3-1: 1つのステートメントを含む `main`関数宣言
</span>

関数定義もステートメントです。前の例全体が単独の文です。

ステートメントは値を返しません。したがって、次のコードが試みるように、`let`文を別の変数に代入することはできません。 エラーが表示されます。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let x = (let y = 6);
}
```

このプログラムを実行すると、次のようなエラーが表示されます。

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^
  |
  = note: variable declaration using `let` is a statement
```

`let y = 6`ステートメントは値を返さないので、`x`が束縛するものはありません。これは、代入が代入の値を返すCやRubyなど、他の言語で行われる処理とは異なります。これらの言語では、`x = y = 6`と書いて、`x`と`y`の両方に`6`という値をつけることができます。これはRustの場合できません。

式は何かに評価され、Rustで書く残りのコードのほとんどを占めます。値'11'に評価される式である'5 + 6'のような単純な数学演算を考えてみましょう。式は文の一部です。リスト3-1では`let y = 6;`という文の`6`は値`6`に評価される式です。関数の呼び出しは式です。マクロの呼び出しは式です。新しいスコープ`{}`を作成するために使用するブロックは、次のような式です。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

この式は、

```rust,ignore
{
    let x = 3;
    x + 1
}
```

ブロックであり、この場合は`4`と評価されます。その値は`let`ステートメントの一部として`y`に束縛されます。最後にセミコロンを付けない`x + 1`行に注意してください。これはこれまで見たほとんどの行とは異なります。式には終了セミコロンは含まれません。式の最後にセミコロンを追加すると、ステートメントに変換され、ステートメントに値が返されません。次に関数の戻り値と式を調べるときは、このことを覚えておいてください。

### 戻り値を持つ関数

関数はそれらを呼び出すコードに値を返すことができます。戻り値の名前は付けませんが、矢印の後に型を宣言します(`->`)。 Rustでは関数の戻り値は、関数本体のブロック内の最終式の値と同義です。`return`キーワードを使用して値を指定することで、関数から早期に返ることができますが、ほとんどの関数は最後の式を暗黙的に返します。次に、値を返す関数の例を示します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

`five`関数には関数呼び出しもマクロも`let`文もありません。それは単に`5`だけです。これは、Rustの完全に有効な機能です。 関数の戻り値の型も`-> i32`と指定されていることに注意してください。このコードを実行してみてください。出力は次のようになります。

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/functions`
The value of x is: 5
```

`5`の`5`は関数の戻り値です。戻り値の型は `i32`です。これをより詳細に調べてみましょう。最初に、`let x = five();`という行は関数の戻り値を使って変数を初期化していることを示しています。関数`five`は`5`を返すので、その行は次のようになります。

```rust
let x = 5;
```

第2に`five`関数はパラメータを持たず戻り値の型を定義しますが、関数の本体はセミコロンを持たない単独の`5'です。

別の例を見てみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

このコードを実行すると`The value of x is is：6`が出力されます。しかし、`x + 1`を含む行の最後にセミコロンを置いて式から文に変更すると、エラーが発生します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

このコードをコンパイルすると、次のようなエラーが発生します。

```text
error[E0308]: mismatched types
 --> src/main.rs:7:28
  |
7 |   fn plus_one(x: i32) -> i32 {
  |  ____________________________^
8 | |     x + 1;
  | |          - help: consider removing this semicolon
9 | | }
  | |_^ expected i32, found ()
  |
  = note: expected type `i32`
             found type `()`
```

メインのエラーメッセージ"mismatched types,"では、このコードの中核的な問題が明らかになります。関数`plus_one`の定義では`i32`が返されますが、ステートメントは評価されません。この値は空のタプル `()`で表されます。したがって、何も返されず、関数定義と矛盾し、エラーが発生します。この出力では、Rustはこの問題を解決するためのメッセージを提供します。セミコロンを削除することで、このエラーは修正できます。
