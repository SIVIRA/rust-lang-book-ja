## 制御フロー

条件が真であるかどうかによっていくつかのコードを実行するかどうかを決定し、条件が真である間に何度かのコードを繰り返し実行することを決定することは、ほとんどのプログラミング言語の基本的なビルディングブロックです。Rustコードの実行フローを制御できる最も一般的な構造は、式とループです。

### `if`式

`if`式は、条件に応じてコードを分岐することを可能にします。条件を指定し「この条件が満たされている場合は、このコードブロックを実行します。条件が満たされない場合は、このコードブロックを実行しないでください」ということを行います。

*projects*ディレクトリに*branches*という名前の新しいプロジェクトを作成して、`if`式を試します。*src/main.rs*ファイルに、次のように入力します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

すべての`if`式はキーワード`if`で始まり、その後に条件が続きます。この場合、変数`number`の値が5未満であるかどうかをチェックします。条件が真である場合に実行したいコードのブロックは、中括弧の中の条件の直後に配置されます。`if`式の条件に関連するコードブロックは、第2章で説明した`match`式のように、*arms*と呼ばれることがあります。

オプションで条件をfalseに評価した場合に実行する別のコードブロックをプログラムに与えるためにここで使用した`else`式を含めることもできます。`else`式を指定せず条件がfalseの場合、プログラムは`if`ブロックをスキップして次のコードに移ります。

このコードを実行してみてください。 次の出力が表示されます。

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was true
```

`number`の値を`false`という条件に変更して何が起こるかを見てみましょう。

```rust,ignore
let number = 7;
```

プログラムをもう一度実行し、出力を確認します。

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was false
```

このコードの条件は*bool*でなければなりません。条件が`bool`でなければエラーになります。例えば、以下の例を見てみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let number = 3;

    if number {
        println!("number was three");
    }
}
```

今回の `if`条件は`3`の値になり、Rustはエラーを投げます。

```text
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |     if number {
  |        ^^^^^^ expected bool, found integral variable
  |
  = note: expected type `bool`
             found type `{integer}`
```

このエラーはRustが`bool`を期待していたのに整数を持っていることを示しています。RubyやJavaScriptなどの言語とは異なり、Rustはブール型以外の型をブール型に自動的に変換しようとはしません。明示的でなければならず、常に条件として`if`をブール値で与えなければなりません。たとえば`if`コードブロックが`0`と等しくないときにのみ実行されるようにするには、`if`式を次のように変更します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let number = 3;

    if number != 0 {
        println!("number was something other than zero");
    }
}
```

このコードを実行すると、 `number was something other than zero`と出力されます。

#### `else if`による複数の条件の処理

`else if`式に` if`と `else`を組み合わせることで、複数の条件を満たすことができます。例えば、以下の例を見てみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

このプログラムには4つの可能なパスがあります。実行後、次の出力が表示されます。

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
number is divisible by 3
```

このプログラムが実行されると各if式を順にチェックし、条件が真である最初のボディを実行します。6が2で割り切れるにもかかわらず、出力は `number is 2 is by divisible by 2`ではなく、`number is divisible by 3`となります。これは、Rustが最初の真の条件のブロックのみを実行し、見つかったら残りの部分をチェックしないからです。

あまりにも多くの`else if`式を使用するとコードが乱雑になる可能性があります。複数ある場合は、コードをリファクタリングすることができます。第6章では、これらの場合に`match'と呼ばれる強力なRust分岐構造を説明します。

#### `let`文で` if`を使う

`if`は式なので、リスト3-2のように`let`ステートメントの右側でそれを使うことができます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

<span class="caption">リスト 3-2: `if`式の結果を変数に代入する</span>

`number`変数は、`if`式の結果に基づいて値にバインドされます。このコードを実行すると、何が起こるかを確認できます。

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/branches`
The value of number is: 5
```

コードブロックはそれらの中の最後の式に評価され数字だけでも式であることを覚えておいてください。この場合、`if`式全体の値はどのコードブロックが実行されるかによって決まります。つまり、`if`の各armの結果となる可能性のある値は同じ型でなければなりません。リスト3-2の`if`アームと`else`アームの両方の結果は`i32`の整数でした。次の例のように型が一致しない場合、エラーが発生します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let condition = true;

    let number = if condition {
        5
    } else {
        "six"
    };

    println!("The value of number is: {}", number);
}
```

このコードをコンパイルしようとすると、エラーが発生します。`if`と`else`には互換性のない値の型があり、Rustはプログラムのどこで問題があったのかを示します。


```text
error[E0308]: if and else have incompatible types
 --> src/main.rs:4:18
  |
4 |       let number = if condition {
  |  __________________^
5 | |         5
6 | |     } else {
7 | |         "six"
8 | |     };
  | |_____^ expected integral variable, found &str
  |
  = note: expected type `{integer}`
             found type `&str`
```

`if`ブロックの式は整数に評価され、`else`ブロックの式は文字列に評価されます。変数は単一の型でなければならないため、これは機能しません。Rustはコンパイル時に`number`変数がどのタイプのものであるかを知る必要があります。コンパイル時に`number`を使用するあらゆる場所でそのタイプが有効であることを検証できます。`number 'の型が実行時にのみ決定された場合、Rustはそれを行うことができません。コンパイラはより複雑になり、任意の変数に対して複数の仮説型を追跡しなければならない場合、コードの保証が少なくなります。


### ループによる繰り返し

1つのコードブロックを複数回実行すると便利なことがよくあります。このタスクでは、Rustはいくつかの*ループ*を提供します。ループはループ本体内のコードを最後まで実行した後、最初からすぐに開始します。ループを試すには、*loop*という新しいプロジェクトを作ってみましょう。 Rustにはloop、while、forという3種類のループがあります。それぞれを試してみましょう。

#### `loop`を使ってコードを繰り返す

`loop`キーワードは、Rustにコードのブロックを何度も何度も何度も何度も実行するよう指示します。

たとえば、*loop*ディレクトリの*src/main.rs*ファイルを次のように変更します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
fn main() {
    loop {
        println!("again!");
    }
}
```

このプログラムを実行すると、プログラムを手動で停止するまで、`again!`が何度も何度も繰り返し表示されます。ほとんどの端末では、キーボードショートカット<span class="keystroke">ctrl-c</span>がサポートされており、連続したループで停止しているプログラムを停止します。

試してみましょう。

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29 secs
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

`^C`の記号は<span class="keystroke">ctrl-c</ span>を押した場所を表します。`^C`の後ろに`again!`という単語が表示される場合と表示されない場合があります。停止信号を受け取ったときのループのどこにコードがあるかによって異なります。

Rustはループから抜け出すための信頼性の高いもう1つの方法を提供します。ループ内で`break`キーワードを使用して、ループの実行をいつ停止するかをプログラムに指示できます。ユーザーが正しい数を推測してゲームに勝ったときにプログラムを終了するには、第2章の「正しい予測の後に終了する」セクションの数あてゲームでこれを実行したことを思い出してください。

#### `loop`からの戻り

`loop`の使用法の1つは、スレッドがそのジョブを完了したかどうかを確認するなど、失敗する可能性のある操作を再試行することです。ただし、その操作の結果をコードの残りの部分に渡す必要があるかもしれません。ループを停止するために使用する`break`式にそれを追加すると、それは`break`したループによって返されます。

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    assert_eq!(result, 20);
}
```

#### `while`による条件付きループ

プログラムがループ内の状態を評価することはしばしば有用です。条件が真である間、ループが実行されます。条件が真でなくなると、プログラムは`break`を呼び出してループを停止します。このループ型は `loop`、`if`、 `else`、`break`の組み合わせを使って実装できます。

しかし、このパターンは非常に一般的なのでRustにはwhileループと呼ばれるビルトインの言語構造があります。リスト3-3は`while`を使い3回ループし、毎回カウントダウンしてから、ループの後に別のメッセージを出力して終了します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

<span class="caption">リスト 3-3: 条件が真である間、`while`ループを使ってコードを実行する</span>

これにより `loop`、`if`、 `else`、`break`を使用した場合に必要となるネストをなくし、より明確になります。条件が成立している間はコードが実行されます。それ以外の場合はループを終了します。

#### `for`を使ってコレクションをループする

配列のようなコレクションの要素をループするために`while`構文を使うことができます。たとえば、リスト3-4を見てみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index = index + 1;
    }
}
```

<span class="caption">リスト 3-4: `while`ループを使ってコレクションの各要素をループする</span>

ここでコードは配列内の要素をカウントアップします。それはインデックス`0`で始まり、配列の最後のインデックスに到達するまで（つまり、`index < 5`が真でなくなるまで）ループします。このコードを実行すると、配列のすべての要素が出力されます。

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
     Running `target/debug/loops`
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```

期待どおり、5つの配列値がすべて端末に表示されます。ある時点で6番目の値をフェッチしようとする前にループは実行を停止します。

しかし、このアプローチはエラーが起こりやすいです。インデックスの長さが間違っていると、プログラムが`panic!`になる可能性があります。コンパイラーはループを介したすべての反復ですべての要素の条件チェックを実行するためのランタイムコードを追加するため処理速度も遅くなります。

より簡潔な方法として、`for`ループを使用してコレクション内の各項目に対していくつかのコードを実行することができます。`for`ループはリスト3-5のコードのようになります。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

<span class="caption">リスト 3-5: `for`ループを使ってコレクションの各要素をループする</span>

このコードを実行すると、コードリスト3-4と同じ出力が表示されます。さらに重要なのはコードの安全性を高めたことで、いくつかのアイテムが見つからないことに起因するバグの可能性を排除したことです。

例えば、リスト3-4のコードでは、配列 `a`から項目を削除したが、条件を`while index <4`に更新するのを忘れた場合、コードは`panic!`に陥ります。`for`ループを使用すると、配列内の値の数を変更した場合、他のコードを変更する必要はありません。

`for`ループの安全性と簡潔さは、それらをRustの中で最も一般的に使用されるループ構造にします。リスト3-3の`while`ループを使ったカウントダウンの例のように、ある程度のコードをいくつか実行したい場合でも、ほとんどの錆びた人は`for`ループを使います。これを行う方法は、標準ライブラリによって提供されるタイプで、ある数字から順番にすべての数字を生成し、別の数字より前に終了するタイプの`Range`を使用することです。

次の例では`for`ループとまだ説明していないもう一つのメソッド`rev`を使ってカウントダウンがどのようになるかを示します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```


## まとめ

変数、スカラーや複合データ型、関数、コメント、`if`式、ループについて学びました。この章で説明するコンセプトで練習する場合は、以下のことを行うプログラムを作成してみてください。

* 気温を華氏と摂氏の間で変換します。
* n番目のフィボナッチ数を生成する。
* 歌の繰り返しを利用して、クリスマスキャロル「The Twelve Days of Christmas」の歌詞を印出力します。

他のプログラミング言語には存在*しない*所有権というRustの概念について説明します。