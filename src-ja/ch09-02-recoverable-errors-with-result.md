## Resultで回復可能なエラー

多くのエラーは、プログラムを完全にストップさせるほど深刻ではありません。時には関数が失敗すると容易に解釈し、対応できるからです。例えば、ファイルを開こうとしてファイルが存在しないために処理が失敗した場合、プロセスを殺すのではなくファイルを作成することができます。

第2章の「`Result`型による潜在的な障害の処理」で`Result`列挙型が以下のように、OkとErrの2値からなるよう定義されていることを思い出してください。

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-the-result-type

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T`と`E`はジェネリック型のパラメータです。ジェネリックについては第10章で詳しく説明します。今知る必要があるのは、`T`が成功した時に`Ok`バリアントに含まれて返される値の型を表すことと、`E`は失敗した時に`Err`バリアントに含まれて返されるエラーの型を表すことです。`Result`はこのようなジェネリックな型引数を含むので、 標準ライブラリ上に定義されている`Result`型や関数などを、成功した時とエラーの値が異なるような様々な場面で使用できるのです。

関数が失敗する可能性があるために`Result`値を返す関数を呼び出しましょう。リスト9-3では、ファイルを開こうとしています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

<span class="caption">リスト 9-3: ファイルを開く</span>

`File::open`が`Result`を返すことをどうやって知ることができるでしょうか？ [標準ライブラリAPIドキュメント](https://doc.rust-lang.org/std/)<!--ignore-->を見るか、コンパイラに尋ねることでできます。関数の戻り値ではない型注釈を与えてコンパイルしようとすると、コンパイラはその型が一致しないことを知らせます。エラーメッセージは`f`の型が何であるかを教えてくれます。`File::open`の戻り値の型は`u32`型ではないので、`let f`文をこれに変更しましょう。

```rust,ignore
let f: u32 = File::open("hello.txt");
```

コンパイルしようとすると、以下のように出力されます。

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
             found type `std::result::Result<std::fs::File, std::io::Error>`
```

これは `File::open`関数の戻り値の型が`Result<T, E>`であることを示しています。ジェネリックパラメータ`T`は成功値の型`std::fs::File`で埋められています。これはファイルハンドルです。エラー値に使用される`E`の型は`std::io::Error`です。

この戻り値の型は`File::open`の呼び出しが成功し、読み書きできるファイルハンドルを返すことを意味します。また関数呼び出しは失敗する可能性もあります。たとえば、ファイルが存在しないか、ファイルにアクセスする権限がないなどです。`File::open`関数には、成功したか失敗したかを知らせる方法と、同時にファイルハンドルまたはエラー情報を与える方法が必要です。この情報は正確に`Result`列挙型が伝えるものです。

`File::open`が成功した場合、変数`f`の値はファイルハンドルを含む`Ok`のインスタンスになります。失敗した場合、`f`の値は発生したエラーの種類に関するより多くの情報を含む`Err`のインスタンスになります。

リスト9-3のコードに`File::open`が返す値に応じて異なるアクションを取るための処理を追加する必要があります。リスト9-4は、基本的なツールを使用して`Result`を処理する方法の1つ、第6章で説明した`match`式を示しています。

<span class="filename">ファイル名: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

<span class="caption">リスト 9-4: `match`式を使用して返却される可能性のあるResult列挙子を処理する</span>

`Option`列挙型のように、`Result`列挙型とそのバリアントは、初期化処理でインポートされているので、`match`arm内で`Ok`と`Err`バリアントの前に`Result::`を指定する必要がないことに注目してください。

ここでは、結果が`Ok`であるとき、内部の`file`値を`Ok`バリアントから戻し、そのファイルハンドル値を変数`f`に代入することをRustに伝えます。`match`の後で、ファイルハンドルを読み書きのために使うことができます。

`match`のもう一方のarmは`File::open`から`Err`値を取得する場合を扱います。この例では、`panic!`マクロを呼び出すことにしました。現在のディレクトリに*hello.txt*という名前のファイルがなく、このコードを実行すると、`panic!`マクロから次の出力が表示されます。

```text
thread 'main' panicked at 'There was a problem opening the file: Error { repr:
Os { code: 2, message: "No such file or directory" } }', src/main.rs:9:12
```

いつものように、この出力は何が間違っているかを正確に教えてくれます。

### 色々なエラーにマッチする

リスト9-4のコードは`File::open`が失敗した理由にかかわらず`panic!`になります。ファイルが存在しないために`File::open`が失敗した場合、ファイルを作成して新しいファイルにハンドルを戻したいと考えています。ファイルを開く権限がないなどの理由で`File::open`が他の理由で失敗した場合、リスト9-4と同じ方法でコードに`panic!`を残しておきます。リスト9-5を見てください。これは、`match`に別のarmを追加します。

<span class="filename">ファイル名: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Tried to create file but there was a problem: {:?}", e),
            },
            other_error => panic!("There was a problem opening the file: {:?}", other_error),
        },
    };
}
```

<span class="caption">リスト 9-5: 色々な種類のエラーを異なる方法で扱う</span>

`File::open`が`Err`バリアントの中で返す値の型は`io::Error`です。これは標準ライブラリによって提供される構造体です。この構造体には`io::ErrorKind`値を得るために呼び出すことができる`kind`メソッドがあります。列挙型`io::ErrorKind`は、標準ライブラリによって提供され、`io`操作の結果として生じるかもしれないさまざまな種類のエラーを表すバリアントを持っています。使用するバリアントは`ErrorKind::NotFound`です。これは開こうとしているファイルがまだ存在しないことを示します。そのため、`f`に`match`しますが、error.kind()に内部の`match`もあります。

マッチガードでチェックしたい条件は、`error.kind()`によって返された値が`ErrorKind`列挙型の`NotFound`バリアントかどうかです。そうであれば、`File::create`でファイルを作成しようとします。しかし、`File::create`も失敗する可能性があるので、内部の`match`式も追加する必要があります。ファイルを開くことができない場合は、別のエラーメッセージが表示されます。外側の`match`の最後のarmは同じままなので、プログラムには欠落しているファイルエラー以外のエラーが発生します。

それはたくさんの`match`です。`match`は非常に強力ですが、非常にプリミティブです。第13章では、クロージャについて学びます。`Result<T、E>`型はクロージャを受け入れる多くのメソッドを持ち、`match`式として実装されています。より熟練のRust開発者がこれを書くかもしれません。

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").map_err(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Tried to create file but there was a problem: {:?}", error);
            })
        } else {
            panic!("There was a problem opening the file: {:?}", error);
        }
    });
}
```

第13章を読んだ後、この例に戻って、標準ライブラリのドキュメントで`map_err`と`unwrap_or_else`メソッドが何をするのか調べてみましょう。エラーを処理するときに、巨大なネストされた`match`式をクリーンアップすることができるこれらのメソッドの多くがあります。

### エラー時にパニックするショートカット: unwrapとexpect

`match`を使うとうまくいきますが、少し冗長になり、必ずしも意図をうまく伝えるとは限りません。`Result<T、E>`型は、さまざまなタスクを実行するために多くのヘルパーメソッドが定義されています。`unwrap`と呼ばれるメソッドの1つは、リスト9-4で書いた`match`式のように実装されたショートカットメソッドです。`Result`値が`Ok`バリアントである場合、`unwrap`は`Ok`の値を返します。`Result`が`Err`バリアントの場合、`unwrap`は`panic!`マクロを呼び出します。実際の`unwrap`の例を以下に示します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

*hello.txt*ファイルなしでこのコードを実行すると、`unwrap`メソッドが行う`panic!`呼び出しからのエラーメッセージが表示されます。

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
src/libcore/result.rs:906:4
```

`unwrap`に似ているもう一つの`expect`メソッドは、`panic!`エラーメッセージを選択することもできます。`unwrap`の代わりに`expect`を使い、良いエラーメッセージを提供することで意図を伝え、パニックの原因を簡単に追跡することができます。 `expect`の構文は次のようになります。

<span class="filename">ファイル名: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

`unwrap`と同じ方法で`expect`を使うことで、ファイルハンドルを返すか`panic!`マクロを呼び出します。`panic!`の呼び出しで`expect`によって使用されるエラーメッセージは`unwrap`が使うデフォルトの`panic!`メッセージではなく`expect`に渡すパラメータになります。以下はその様子です。

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }', src/libcore/result.rs:906:4
```

このエラーメッセージは、指定したテキスト`Failed to open hello.txt`で始まるので、このエラーメッセージがどこから来たのかを簡単に見つけることができます。複数の場所で`unwrap`を使用すると、`unwrap`が同じメッセージを表示するすべての`unwrap`を呼び出すため、`unwrap`がパニックを起こしていることを正確に把握するのに時間がかかることがあります。

### エラーを委譲する

失敗する可能性のある何かを呼び出す実装をした関数を書く際、関数内でエラーを処理する代わりに、呼び出し元がどうするかを決められるようにエラーを返すことができます。これはエラーの*委譲*として認知され、呼び出し元に自分のコードの文脈で利用可能なものよりも、エラーの処理法を規定する情報やロジックがより多くある箇所を制御してもらいます。

たとえば、コードリスト9-6は、ファイルからユーザー名を読み取る関数を示しています。ファイルが存在しないか読み取れない場合、この関数は、この関数を呼び出したコードにこれらのエラーを返します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

<span class="caption">リスト 9-6: `match`でエラーを呼び出し元のコードに返す関数</span>

この関数ははるかに短い方法で記述することができますが、エラー処理を探索するために手動で多くの作業を行うことから始めます。最後に、簡単な方法を示します。最初に関数の戻り値の型を見てみましょう。`Result<String, io::Error>`です。これは、ジェネリックパラメータ`T`が具体型`String`で埋められ、ジェネリック型`E`が具体的型`io::Error`の場合に、関数が`Result<T, E>`型の値を返すことを意味します。この関数が問題なく成功した場合、この関数を呼び出すコードは、`String`(この関数がファイルから読み取ったユーザー名)を保持する`Ok`値を受け取ります。この関数が何らかの問題に遭遇した場合、この関数を呼び出すコードは、問題の内容に関する詳細情報を含む `io::Error`のインスタンスを保持する`Err`値を受け取ります。関数の戻り値の型に`io::Error`を選んだのは、この関数本体で呼び出している失敗する可能性のある処理が両方(`File::open`関数と`read_to_string`メソッド)とも偶然この型をエラー値として返すからです。

関数の本体は`File::open`関数を呼び出すことから始まります。リスト9-4の`match`に似た`match`で返された`Result`値を扱い、`Err`の場合に`panic!`を呼び出すのではなく、この関数から早くリターンして、この関数のエラー値として`File::open`からのエラー値を呼び出しコードに返します。`File::open`が成功すると、ファイルハンドルを変数`f`に格納して処理を続行します。

次に、変数`s`に新しい`String`を作成し、`f`のファイルハンドルの`read_to_string`メソッドを呼び出して、ファイルの内容を`s`に読み込みます。`read_to_string`メソッドは、`File::open`が成功したとしても、失敗する可能性があるため、`Result`を返します。`result`を処理するために別の`match`が必要です。もし`read_to_string`が成功すれば関数は成功し、`s`の`Ok`でラップされたファイルからユーザ名を返します。`read_to_string`が失敗した場合、`File::open`の戻り値を処理した`match`にエラー値を返したのと同じ方法でエラー値を返します。しかし、関数の最後の式であるため、`return`を明示的に言う必要はありません。

このコードを呼び出すコードは、ユーザ名を含む`Ok`値か、`io::Error`を含む`Err`値を得て扱います。呼び出し元のコードがそれらの値をどうするかはわかりません。呼び出しコードが`Err`値を得たら、 例えば、`panic!`を呼び出してプログラムをクラッシュさせたり、デフォルトのユーザ名を使ったり、ファイル以外の場所からユーザ名を検索したりできるでしょう。呼び出し元のコードが実際に何をしようとするかについて、十分な情報がないので、成功や失敗情報を全て委譲して適切に扱えるようにするのです。

Rustにおいて、この種のエラー委譲は非常に一般的なので、Rustにはこれをしやすくする`?`演算子が用意されています。

#### エラー委譲のショートカット: ?演算子

リスト9-7は、リスト9-6と同じ機能を持つ`read_username_from_file`の実装を示していますが、この実装では`?`演算子を使います。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

<span class="caption">リスト 9-7: `?`を使って呼び出し元のコードにエラーを返す関数</span>

`Result`値の後に置かれた`?`は、リスト9-6の`Result`値を扱うために定義した`match`式とほぼ同じように動作します。`Result`の値が`Ok`ならば、`Ok`の値がこの式から返され、プログラムは続行されます。値が`Err`の場合、`return`キーワードを使用したかのように関数全体から`Err`が返され、エラー値は呼び出し元のコードに委譲されます。

リスト9-6の`match`式と`?`演算子には違いは、`?`で取られたエラー値は、あるタイプから別のタイプへのエラーを変換するために使用される標準ライブラリの`From`特性で定義された`from`関数を通って行きます。`?`が`from`関数を呼び出すと、受け取ったエラーの型は、現在の関数の戻り値の型で定義されているエラーの型に変換されます。これは、関数が多くの異なる理由で失敗する可能性がある場合でも、関数が失敗する可能性のあるすべての方法を表す1つのエラータイプを返す場合に便利です。各エラータイプが`from`関数を実装して、返されたエラータイプに変換する方法を定義する限り、`?`は変換を自動的に処理します。

リスト9-7の文脈では、`File::open`呼び出しの最後の`?`は、`Ok`の値を変数`f`に返します。エラーが発生した場合、`?`は関数全体からすぐにリターンし、`Err`値を呼び出しコードに与えます。`read_to_string`呼び出しの最後の`?`も同じことが言えます。

`?`演算子は多くの定型文を削除し、この関数の実装を簡単にします。リスト9-8のように、`?`の直後にメソッド呼び出しを連鎖させることで、このコードをさらに短縮することもできます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

<span class="caption">リスト 9-8: `?`演算子の後のメソッド呼び出しを連結する</span>

新しい`String`の作成を`s`の中で関数の始めに移しました。その部分は変更されていません。変数`f`を作成するのではなく、`File::open("hello.txt")?`の結果に`read_to_string`の呼び出しを直接連結しました。`read_to_string`呼び出しの最後に`?`があります。エラーを返すのではなく、`File::open`と`read_to_string`の両方が成功したときに`s`のユーザ名を含む`Ok`値を返します。この機能はリスト9-6とリスト9-7と同じです。ただ単に異なるバージョンのよりプログラマフレンドリーな書き方なのです。

この関数を書くさまざまな方法について言えば、これをもっと短くする方法があります。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::io;
use std::fs;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

<span class="caption">リスト 9-9: `fs::read_to_string`を使う</span>

ファイルを文字列に読み込むのはかなり一般的な操作なので、Rustは`fs::read_to_string`という便利な関数を提供します。これはファイルを開き、新しい`String`を作成し、ファイルの内容を読み込み、内容を`String`に入れて返します。もちろん、これはこのエラー処理の全てを見せる機会を与えるものではないので、最初の方法としては難しいものでした。

#### `?`演算子は、`Result`を返す関数でしか使用できない

`?`演算子は、リスト9-6で定義した`match`式と同じように動作するように定義されているので、`Result`の戻り値型を持つ関数でのみ使用できます。`result`の戻り値の型を必要とする`match`の部分は`return Err(e)`なので、関数の戻り値の型はこの`return`と互換性がある`Result`でなければなりません。

`main`関数で`?`演算子を使用したらどうなるか見てみましょう。main関数は戻り値が`()`でした。

```rust,ignore
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

このコードをコンパイルすると、以下のようなエラーメッセージが得られます。

```text
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `std::ops::Try`)
 --> src/main.rs:4:13
  |
4 |     let f = File::open("hello.txt")?;
  |             ^^^^^^^^^^^^^^^^^^^^^^^^ cannot use the `?` operator in a function that returns `()`
  |
  = help: the trait `std::ops::Try` is not implemented for `()`
  = note: required by `std::ops::Try::from_error`
```

このエラーは、`?`演算子は`Result`を返す関数でしか使用が許可されないと指摘しています。`Result`を返さない関数では、`Result`を返す別の関数を呼び出した時、`?`演算子を使用してエラーを呼び出し元に委譲する可能性を生み出す代わりに、`match`か`Result`のメソッドのどれかを使う必要があるでしょう。

しかし、 `main`関数は`Result<T,E>`を返すことができます。

```rust,ignore
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())

```

`Box<dyn Error>`は"trait object"と呼ばれます。これについては第17章で説明します。今のところ、`Box<dyn Error>`は"あらゆる種類のエラー"を意味します。

ここまでで`panic!`呼び出しや`Result`を返す詳細について議論し終えたので、 どんな場合にどちらを使うのが適切か決める方法についての話に戻りましょう。