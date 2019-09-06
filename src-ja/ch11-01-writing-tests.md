## テストの記述法

テストは、非テストコードが期待どおりに機能していることを検証するRust関数です。テスト関数の本体は、通常次の3つのアクションを実行します。

1. 必要なデータや状態をセットアップする
2. テスト対象のコードを実行する
3. 結果が想定通りかアサーションする

Rustが特にこれらの動作を行うテストを書くために用意している機能を見ていきましょう。これには、`test`属性、いくつかのマクロ、`should_panic`属性が含まれます。

### テスト関数の解剖

最も簡単な方法では、Rustでのテストは`test`属性で注釈された関数です。属性はRustコードに関するメタデータです。1つの例は、第5章で構造体で使用した`derive`属性です。関数をテスト関数に変更するには、`fn`の前の行に`＃[test]`を追加します。`cargo test`コマンドでテストを実行すると、Rustは`test`属性でアノテーションされた関数を実行し、各テスト関数が成功するか失敗するかを報告するテスト用バイナリを作成します。

Cargoで新しいライブラリプロジェクトを作成すると、テスト機能付きのテストモジュールが自動的に生成されます。このモジュールはテストの作成を開始するのに役立ちますので、新しいプロジェクトを開始するたびにテスト関数の構造と構文を正確に調べる必要はありません。必要なだけ多くのテスト機能とテストモジュールを追加することができます。

実際にコードをテストすることなく、私たちのために生成されたテンプレートテストを試すことで、テストの仕組みのいくつかの側面を探求します。次に、私たちが書いたコードを呼び出し、その動作が正しいことを主張する実際のテストを書きます。

`adder`という新しいライブラリプロジェクトを作成しましょう。


```text
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

`adder`ライブラリの*src/lib.rs*ファイルの内容はリスト11-1のようになります。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

<span class="caption">リスト 11-1: `cargo new`で自動生成されたテストモジュールと関数</span>

今のところ、上の2行を無視して、関数にどのように作用するかを見てみましょう。`fn`行の前の`#[test]`注釈に注意してください。この属性はこれがテスト関数であることを示しているので、テストランナーはこの関数をテストとして扱うことを知っています。また、`tests`モジュールに非テスト関数を組み込んで、共通のシナリオを設定したり、一般的な操作を行うこともできます。したがって、`#[test]`属性を使ってどの関数がテストであるかを示す必要があります。

関数本体は `assert_eq!`マクロを使用して、2 + 2が4に等しいことを宣言します。このアサーションは、典型的なテストの形式の例として役立ちます。このテストが合格することを確認するために実行してみましょう。

`cargo test`コマンドは、リスト11-2に示すように、プロジェクト内のすべてのテストを実行します。

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

<span class="caption">リスト 11-2: 自動生成されたテストを走らせた出力</span>

Cargoはコンパイルされ、テストが実行されました。`Compiling`、`Finished`、`Running`行の後に`running 1 test`という行があります。次の行には、生成されたテスト関数の名前、`it_works`とそのテストを実行した結果`ok`が表示されます。次に、テストの実行の概要が表示されます。`test result：ok.`というテキストは、すべてのテストが成功したことを意味します。 `0 failed`は合格または不合格のテストの数を合計します。

無視されたテストはありませんので、サマリーには`0 ignored`と表示されます。また、実行されているテストをフィルタリングしていないため、サマリーの最後に`0 filtered out`と表示されます。テストの無視と除外については、次のセクション「テストの実行され方を制御する」で説明します。

`0 measured`という統計は、性能を測定するベンチマーク試験のためのものである。ベンチマークテストは、この記事の執筆時点でNightlyのRustだけで利用可能です。詳細は[ベンチマークテストのドキュメンテーション][bench]を参照してください。

[bench]: ../unstable-book/library-features/test.html

`Doc-tests adder`で始まるテスト出力の次の部分は、ドキュメンテーションテストの結果です。まだドキュメントテストはありませんが、RustはAPIドキュメントに記載されているコード例をコンパイルできます。この機能は、ドキュメントとコードを同期させるのに役立ちます。第14章の「テストとしてのドキュメンテーションコメント」セクションでドキュメンテーション・テストを書く方法について説明します。今は`Doc-tests`の出力を無視します。

テストの名前を変更してテスト出力をどのように変更するかを見てみましょう。`it_works`関数を`exploration`のような別の名前に変更してください。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

その後、再度`cargo test`を実行します。出力に`it_works`の代わりに`exploration`が表示されます。

```text
running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

別のテストを追加しましょう。今回は失敗したテストを行います。テスト機能で何かがパニックすると、テストは失敗します。各テストは新しいスレッドで実行され、メインスレッドがテストスレッドが終了したことがわかると、テストは失敗とマークされます。第9章でパニックを引き起こす最も簡単な方法について話しました。これは`panic!`マクロを呼び出すことです。*src/lib.rs*ファイルがリスト11-3のようになるよう、新しいテストanotherを入力してください。

<span class="filename">ファイル名: src/lib.rs</span>

```rust,panics
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

<span class="caption">リスト 11-3: `panic!`マクロを呼び出したために失敗する2番目のテストを追加する</span>

`cargo test`を使ってテストを再実行してください。出力はリスト11-4のようになります。これは、`exploration`テストが成功し、`another`が失敗したことを示しています。

```text
running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
    thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed
```

<span class="caption">リスト 11-4: 1つのテストが通り、失敗するときのテスト結果</span>

`ok`の代わりに、`test tests::another`行は`FAILED`を表示します。2つの新しいセクションが個々の結果とサマリーの間に表示されます。最初のセクションには、それぞれのテストの失敗の詳細な理由が表示されます。この場合、`another`は*src/lib.rs*ファイルの10行目で`Make this test fail`というパニックになりました。次のセクションでは、失敗したすべてのテストの名前だけを示します。これは、たくさんのテストがあり、多くの詳細な失敗したテスト出力がある場合に便利です。失敗したテストの名前を使用して、そのテストだけを実行してより簡単にデバッグすることができます。テストを実行する方法の詳細については、「テストの実行され方を制御する」のセクションで説明します。

サマリー行が最後に表示されます。全体的に、テスト結果はFAILEDです。1回のテストパスと1回のテストが失敗しました。

さまざまなシナリオでテスト結果がどのように見えるかを見てきたので、テストで有用な`panic!`以外のマクロを見てみましょう。

### `assert!`マクロで結果を確認する

標準ライブラリによって提供される`assert!`マクロは、テスト中のある条件が`true`と評価されるようにしたいときに便利です。`assert!`マクロには、ブール値に評価される引数を与えます。値が`true`の場合、`assert!`は何もせず、テストに合格します。 値が `false`の場合、` assert！ `マクロは` panic！ `マクロを呼び出し、テストを失敗させます。 `assert！`マクロを使うと、私たちのコードが意図した通りに機能していることを確認するのに役立ちます。

第5章のリスト5-15では、リスト11-5で繰り返される`Rectangle`構造体と`can_hold`メソッドを使用しました。このコードを*src/lib.rs*ファイルに置き、`assert!`マクロを使っていくつかのテストを書きましょう。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
#[derive(Debug)]
pub struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

<span class="caption">リスト 11-5: 第5章から`Rectangle`構造体とその`can_hold`メソッドを使用する</span>

`can_hold`メソッドは論理値を返すので、`assert!`マクロの完璧なユースケースになるわけです。リスト11-6で、長さが8、幅が7の`Rectangle`インスタンスを生成し、これが長さ5、幅1の別の`Rectangle`インスタンスを保持できるとアサーションすることでcan_holdを用いるテストを書きます。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(larger.can_hold(&smaller));
    }
}
```

<span class="caption">リスト 11-6:より大きな四角形がより小さな四角形を確かに保持できるかを確認するcan_hold用のテスト</span>

`tests`モジュールの中に新しい行を追加しました。`use super::*;`です。`tests`モジュールは、第7章の「プライバシー規則」で説明した通常の公開ルールに従う普通のモジュールです。`tests`モジュールは内部モジュールなので、外部モジュールのテスト対象コードを内部モジュールのスコープに持っていく必要があります。ここではglobを使用しているので、外部モジュールで定義するものはすべてこの`tests`モジュールで使用できます。

テストに`large_can_hold_smaller`という名前をつけて、必要な二つの`Rectangle`インスタンスを作成しました。それから`assert!`マクロを呼び出し、`greater.can_hold(&smaller)`を呼び出した結果を渡しました。この式は`true`を返すと想定されているので、テストは成功するはずです。確認してみましょう。

```text
running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

このテストは合格します。別のテストを追加してみましょう。今回は、小さい四角形は、より大きな四角形を保持できないことをアサーションします。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(!smaller.can_hold(&larger));
    }
}
```

この場合の`can_hold`関数の正しい結果は`false`なので、`assert!`マクロに渡す前に否定する必要があります。その結果、`can_hold`が`false`を返した場合、テストは成功します。

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

2つのテストが合格しました。次に、コードにバグを導入したときに、テスト結果に何が起こるかを見てみましょう。大文字と小文字を長さを比較する際に小文字の記号に置き換えて、`can_hold`メソッドの実装を変更しましょう。

```rust,not_desired_behavior
# fn main() {}
# #[derive(Debug)]
# pub struct Rectangle {
#     length: u32,
#     width: u32,
# }
// --snip--

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length < other.length && self.width > other.width
    }
}
```

テストを実行すると、以下が生成されます。

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... FAILED

failures:

---- tests::larger_can_hold_smaller stdout ----
    thread 'tests::larger_can_hold_smaller' panicked at 'assertion failed:
    larger.can_hold(&smaller)', src/lib.rs:22:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

このテストではバグが見つかりました。`greater.length`が8で`smaller.length`が5であるため、`can_hold`の長さの比較は`false`を返します。8は5以上だからです。

### `assert_eq!`と`assert_ne!`マクロで等値性をテストする

機能をテストする一般的な方法は、テスト中のコードの結果と、コードが一致することを期待する値とを比較することです。これを行うには、`assert!`マクロを使い、`==`演算子を使って式を渡します。しかし、これは、標準ライブラリが`assert_eq!`と`assert_ne!`マクロのペアを提供し、このテストをより便利に行うような共通のテストです。これらのマクロは、それぞれ等式または不等式の2つの引数を比較します。アサーションに失敗した場合は、2つの値も出力されます。なぜなら、*なぜ*テストが失敗したかを簡単に見ることができるからです。逆に`assert!`マクロは、`==`式の値がfalse値になったことしか示唆せず、`false`値に導いた値は出力しません。

リスト11-7では、`add_two`という名前の関数を作成し、その引数に`2`を加えて結果を返します。次に、この関数を`assert_eq!`マクロを使ってテストします。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

<span class="caption">リスト 11-7: `assert_eq!`マクロで`add_two`関数をテストする</span>

テストが通ることを確認しましょう。

```text
running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

`assert_eq!`マクロに与えた最初の引数`4`は、`add_two(2)`を呼び出した結果と同じです。このテストの行は`test tests::it_adds_two ... ok`であり、`ok`テキストはテストが成功したことを示します。

`assert_eq!`を使ったテストが失敗したときの様子を見てみましょう。`add_two`関数の実装を変更して、代わりに`3`を追加してください。

```rust,not_desired_behavior
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

テストをもう一度実行します。

```text
running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
        thread 'tests::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

テストではバグが見つかりました。`it_adds_two`のテストは失敗し、`assertion failed: '(left == right)'`というメッセージを表示し、`left`は`4`で、`right`は`5`だったと示しています。このメッセージは有用で、デバッグを開始する助けになります。 `assert_eq!`の`left`引数は`4`だったが、`add_two(2)`がある`right`引数は`5`だったことを意味しています。

いくつかの言語やテストフレームワークでは、2つの値が同じであることを宣言する関数のパラメータを`expected`と`actual`と呼び、引数を指定する順序が重要であることに注意してください。しかし、Rustでは、それらは`left`と`right`と呼び、期待する値とテスト対象コードが生成する値を指定する順序は関係ありません。このテストでアサーションを`assert_eq!(add_two(2), 4)`と書くと、`assertion failed:'(left == right)'`と`left`が`5`で`right`が`4`と表示されるわけです。
	
`assert_ne!`マクロは、与えられた2つの値が等しくない場合に合格し、等しい場合に失敗します。このマクロは、価値が何になるかがわからない場合に最も役立ちますが、コードが意図したとおりに機能している場合には絶対に価値がないことを知ります。たとえば、何らかの方法で入力を変更することが保証されている関数をテストする場合、入力を変更する方法はテストを実行する曜日によって異なりますが、関数の出力が入力と等しくないことを示します。

表面上で、`assert_eq!`と`assert_ne!`マクロは`==`と`!=`の演算子をそれぞれ使います。アサーションが失敗すると、これらのマクロは引数を出力します。これは、比較される値が`PartialEq`と`Debug`トレイトを実装しなければならないことを意味します。すべてのプリミティブ型とほとんどの標準ライブラリ型は、これらのトレイトを実装しています。定義する構造体と列挙体の場合、それらの型の値が等しいかどうかを宣言するために`PartialEq`を実装する必要があります。アサーションが失敗したときに値を出力するには `Debug`を実装する必要があります。第5章のリスト5-12で述べたように、どちらのトレイトも継承可能トレイトなので、これは構造体または列挙型の定義に`#[derive(PartialEq, Debug)]`アノテーションを追加するのと同じくらい簡単です。これらおよび他の継承可能トレイトの詳細については、付録C「継承可能トレイト」を参照してください。

### カスタムの失敗メッセージを追加する

また、 `assert!`、`assert_eq!`、`assert_ne!`マクロにオプションの引数として失敗メッセージとともに出力されるカスタムメッセージを追加することもできます。`assert!`に対する必須の引数の1つまたは`assert_eq!`と`assert_ne!`の2つの必須の引数の後に指定された引数は`format!`マクロに渡され (format!マクロについては第8章の「`+`演算子または、`format!`マクロで連結する」節で議論しました)、 `{}`のプレースホルダと値を含む書式文字列を渡すことができます。カスタムメッセージは、アサーションが意味するものを明文化するのに便利です。テストが失敗した場合は、コードの問題の詳細を知ることができます。

たとえば、名前で人で挨拶する関数があり、関数に渡す名前が出力に現れることをテストしたいとしましょう。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```

このプログラムの必要事項はまだ合意が得られておらず、挨拶の先頭の`Hello`というテキストは変わるだろうということは確かです。要件が変わった時にテストを更新しなくてもよいようにしたいと決定したので、`greeting`関数から返る値と正確な等値性を確認するのではなく、出力が入力引数のテキストを含むことをアサーションするだけにします。

`greeting`が`name`を含まないように変更して、このテストの失敗を見てみましょう。

```rust,not_desired_behavior
# fn main() {}
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

このテストを実行すると、以下が出力されます。

```text
running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'assertion failed:
result.contains("Carol")', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::greeting_contains_name
```

この結果は、アサーションが失敗し、アサーションがどのラインにあるかを示します。この場合、より有用な失敗メッセージは、`greeting`関数から得た値を表示します。テスト関数を変更して、`greeting`関数から得た実際の値で埋められたプレースホルダを持つフォーマット文字列から作られたカスタムエラーメッセージを与えましょう。

```rust,ignore
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

テストを実行すると、より有益なエラーメッセージが表示されます。

```text
---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'Greeting did not
contain name, value was `Hello!`', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

実際にテスト結果に表示された値がわかります。これは、発生すると予想されるものではなく、発生したことをデバッグするのに役立ちます。

### `should_panic`でパニックを確認する

コードが正しい値を返すことを確認することに加えて、コードが予期したとおりにエラー状態を処理することを確認することも重要です。たとえば、第9章のリスト9-10で作成した`Guess`型を考えてみましょう。`Guess`を使う他のコードは`Guess`インスタンスが1から100の間の値しか含まないという保証に依存します。その範囲外の値を持つ`Guess`インスタンスを作成しようとするテストを書くことができます。

これを行うには、もう一つの属性`should_panic`をテスト関数に追加します。この属性は、関数内のコードがパニックになった場合にテストに合格します。関数内のコードがパニックに陥らなければ、テストは失敗します。

リスト11-8は`Guess::new`のエラー状態が次のようになるのを期待していることをチェックするテストを示しています。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">リスト 11-8: 状況が`panic!`を引き起こすとテストする</span>

`#[should_panic]`属性は`#[test]`属性の後で、それが適用されるテスト関数の前に置かれます。このテストに合格したときの結果を見てみましょう。

```text
running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

では、値が100より大きい場合、`new`関数がパニックになるという条件を取り除いて、コードにバグを導入しましょう。

```rust,not_desired_behavior
# fn main() {}
# pub struct Guess {
#     value: i32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1  {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}
```

リスト11-8のテストを実行すると失敗します。

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

この場合は非常に有用なメッセージはありませんが、テスト関数を見ると`#[should_panic]`と注釈されています。テスト機能のコードがパニックを引き起こさなかったことを意味します。

`should_panic`を使ったテストは、コードがある程度のパニックを引き起こしたことだけを示しているので、不正確になる可能性があります。`should_panic`テストは、起こることを期待していたのとは異なる理由でパニックになっても、合格します。 `should_panic`テストをより正確にするために、`should_panic`属性に`expected`パラメータを追加することができます。テストハーネスは、失敗メッセージに指定されたテキストが含まれていることを確認します。 たとえば、リスト11-9の`Guess`の修正されたコードを考えてみましょう。ここで`new`関数は、値が小さすぎるか大きすぎるかによってメッセージが異なります。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
# pub struct Guess {
#     value: i32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!("Guess value must be greater than or equal to 1, got {}.",
                   value);
        } else if value > 100 {
            panic!("Guess value must be less than or equal to 100, got {}.",
                   value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">リスト 11-9: 状況が特定のパニックメッセージで`panic!`を引き起こすことをテストする</span>

`should_panic`属性の`expected`引数に置いた値が`Guess::new`関数がパニックしたメッセージの一部になっているので、 このテストは通ります。予想されるパニックメッセージ全体を指定することもでき、そうすれば今回の場合、`Guess value must be less than or equal to 100, got 200.`となります。`should_panic`の予想される引数に指定すると決めたものは、パニックメッセージの固有性や活動性、テストの正確性によります。今回の場合、パニックメッセージの一部でも、テスト関数内のコードが、`else if value > 100`ケースを実行していると確認するのに事足りるのです。

`expected`メッセージで`should_panic`テストが失敗したときに何が起こるかを知るために、`if value < 1`と`else if value > 100`ブロックの本体を入れ替えて、もう一度コードにバグを導入してみましょう。

```rust,ignore,not_desired_behavior
if value < 1 {
    panic!("Guess value must be less than or equal to 100, got {}.", value);
} else if value > 100 {
    panic!("Guess value must be greater than or equal to 1, got {}.", value);
}
```

今回は`should_panic`テストを実行すると失敗します。

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
        thread 'tests::greater_than_100' panicked at 'Guess value must be
greater than or equal to 1, got 200.', src/lib.rs:11:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
note: Panic did not include expected string 'Guess value must be less than or
equal to 100'

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

失敗メッセージは、このテストが期待通りにパニックになったことを示しますが、予想される文字列の`'Guess value must be less than or equal to 100'`を含んでいませんでした。この場合、私たちが得たパニックメッセージは、`Guess value must be greater than or equal to 1, got 200`となりました。そうしてバグの所在地を割り出し始めることができるわけです。

### テストで `Result<T, E>`を使う

これまでは、失敗したときにパニックするテストを書いてきました。`Result<T, E>`を使ったテストも書くことができます。最初の例がありますが、代わりに結果があります。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

ここでは、`it_works`関数を変更して結果を返します。また、`assert_eq!`ではなく、成功例の場合は`Ok(())`を、失敗の場合は`String`を内部に持つ`Err`を返します。これまでのように、このテストは失敗または成功しますが、パニックに基づくのではなく、`Result<T, E>`を使用してその決定を行います。このため、これらの関数の1つと`#[should_panic]`を使うことはできません。代わりに`Err`を返すべきです。

テストを書くためのいくつかの方法を知ったので、テストを実行し、`cargo test`で使用できるさまざまなオプションを調べるときに何が起きているのかを見てみましょう。
