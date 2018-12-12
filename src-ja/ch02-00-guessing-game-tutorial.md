# 数当てゲームをプログラムする

実践的なプロジェクトを一緒に進めて、Rustに飛び込みましょう＞この章では、実際のプログラムでそれらを使用する方法を示すことによって、いくつかの一般的なRustの概念を紹介します。`let`、`match`、メソッド、関連する関数、外部クレートの使用などについて学習します。以下の章では、これらのアイデアについて詳しく説明します。この章では、基本を練習します。

古典的な初心者プログラミングの問題を実装します。数あてゲームです。これはどのように動作するのでしょうか？プログラムは1から100の間のランダムな整数を生成します。次に、予想を入力するようプレーヤーに促します。予想が入力されると、予想が低すぎるか高すぎるかを示すプログラムが表示されます。予想が正しい場合、ゲームは祝福メッセージを出力して終了します。

## 新しいプロジェクトを設定する

新しいプロジェクトを設定するには、第1章で作成した*projects*ディレクトリに移動し、Cargoを使用して新しいプロジェクトを作成します。

```text
$ cargo new guessing_game
$ cd guessing_game
```

最初のコマンド `cargo new`は、プロジェクトの名前（` guessing_game`）を最初の引数として取ります。2番目のコマンドは新しいプロジェクトのディレクトリに移動します。

生成された*Cargo.toml*ファイルを見てください。

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

Cargoが環境から取得した著者情報が正しくない場合は、ファイルに修正してもう一度保存してください。

第1章でみたように、`cargo new`は"Hello, world!"プログラムを生成します。*src/main.rs*ファイルをチェックしてください。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

今度はこの"Hello, world!"プログラムをコンパイルして、`cargo run`コマンドを使って同じステップで実行しましょう。

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50 secs
     Running `target/debug/guessing_game`
Hello, world!
```

`run`コマンドは、このゲームでやっているように、プロジェクトを迅速に反復する必要があるときに、すぐに各反復をテストして次のものに移る必要があるときに便利です。

*src/main.rs*ファイルを再度開きます。このファイルにすべてのコードを記述します。

## 数あてゲームの処理をする

数あてゲームプログラムの最初の部分は、ユーザーの入力を求め、その入力を処理し、入力が予想される形式であることを確認します。まず、プレイヤーに予想を入力させます。リスト2-1のコードを*src/main.rs*に入力します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<span class="caption">リスト 2-1: ユーザーからの予想を取得して出力するコード</span>

このコードには多くの情報が含まれていますので、行ごとに説明しましょう。ユーザー入力を取得し、その結果を出力として出力するには、 `io`（入出力）ライブラリをスコープに持っていく必要があります。`io`ライブラリは標準ライブラリ（`std`として知られています）からきています。

```rust,ignore
use std::io;
```
デフォルトでは、Rustは[prelude][prelude]<!-- ignore -->内のすべてのプログラムの範囲にわずかなタイプしか持ちません。使用する型がpreludeにない場合は、そのタイプを`use`ステートメントで明示的にスコープに入れる必要があります。`std::io`ライブラリを使うと、ユーザの入力を受け入れることを含む多くの便利な機能が提供されます。

[prelude]: ../../std/prelude/index.html

第1章で見たように、main関数はプログラムへの入り口です。

```rust,ignore
fn main() {
```

`fn`構文は新しい関数を宣言し、`()`は引数がないことを示し、`{`は関数の本体を開始します。

第1章で学んだように、`println!`は文字列を画面に表示するマクロです。

```rust,ignore
println!("Guess the number!");

println!("Please input your guess.");
```

このコードはゲームが何であるかを示すプロンプトを表示し、ユーザーからの入力を要求しています。

### 値を変数に格納する

次に、次のようにユーザー入力を格納する場所を作成します。

```rust,ignore
let mut guess = String::new();
```

これは `let`文であり、*変数*の作成に使用されていることに注目してください。別の例があります。

```rust,ignore
let foo = bar;
```

この行は`foo`という名前の新しい変数を作成し、それを`bar`変数の値にバインドします。Rustでは変数はデフォルトでは不変です。この概念については、第3章の「変数と可変性」のセクションで詳しく説明します。次の例は、変数名の前に`mut`を使用して変数を可変にする方法を示しています。

```rust,ignore
let foo = 5; // 不変
let mut bar = 5; // 可変
```

> 注：//構文は、行末まで続くコメントを開始します。Rustはコメントのすべてを無視します。
> これについては第3章で詳しく説明します。

次の行は`let mut guess`が`guess`という変数を定義します。等号(`=`)の反対側には、`guess`がバインドされている値があります。これは`String::new`を呼び出す結果で、`String`の新しいインスタンスを返す関数です。[`String`][string]<!-- ignore -->は標準ライブラリが提供する文字列型で、拡張可能なUTF-8でエンコードされたテキストです。

[string]: ../../std/string/struct.String.html

`::new`行の`::`構文は、`new`が`String`型の*関連関数*であることを示します。関連関数は、Stringの特定のインスタンスではなく、型(この場合は`String`)で実装されます。いくつかの言語はこれを*staticメソッド*と呼びます。

この`new`関数は、空の新しい文字列を作成します。多くの型の中に`new`関数があります。なぜなら、ある種の新しい値を作る関数の一般的な名前だからです。

要約すると、 `let mut guess = String::new();`行は現在、`String`の新しい空のインスタンスにバインドされている変数を作成しました。

プログラムの最初の行に`use std::io;`という標準ライブラリの入出力機能が含まれていることを思い出してください。ここで`io`に関連する関数`stdin`を呼び出します：

```rust,ignore
io::stdin().read_line(&mut guess)
    .expect("Failed to read line");
```

もしプログラムの先頭に`use std::io`行がなければ、この関数呼び出しを`std::io::stdin`と書くことができました。`stdin`関数は、端末の標準入力へのハンドルを表す型である[`std::io::Stdin`][iostdin]<!-- ignore -->のインスタンスを返します。

[iostdin]: ../../std/io/struct.Stdin.html

コードの次の部分である`.read_line(&mut guess)`は、ユーザからの入力を得るために標準入力ハンドルの[read_line][read_line]<!-- ignore -->メソッドを呼び出します。`read_line`の引数に`&mut guess`を渡しています。

[read_line]: ../../std/io/struct.Stdin.html#method.read_line

`read_line`の役割は、ユーザが標準入力に入力したものをそのまま文字列に入れて、その文字列を引数として取ることです。文字列引数は変更可能である必要があり、メソッドはユーザー入力を追加して文字列の内容を変更できます。

`&`は、この引数が*参照*であることを示します。コードの複数の部分がそのデータを複数回メモリにコピーする必要なしに1つのデータにアクセスできるようにします。参照は複雑な機能であり、Rustの主な利点の1つは、参照を使用することがどれほど安全で簡単なのかです。このプログラムを終了するために多くの詳細を知る必要はありません。この時点で知る必要があるのは、同じ変数の場合、参照はデフォルトでは不変であるということです。したがって、`&guess`ではなく`＆mut guess`を書き換えて変更可能にする必要があります（第4章では、参照をより詳しく説明します）。

### `Result`型による潜在的な障害の処理

このコード行ではあまり成し遂げられていません。これまで説明してきたのは1行のテキストですが、これは単一の論理的なコード行の最初の部分にすぎません。2番目の部分はこのメソッドです。

```rust,ignore
.expect("Failed to read line");
```

`.foo()`構文でメソッドを呼び出すときには、長い行を分割するのに役立つ改行やその他の空白を導入することが賢明です。このコードを次のように書くことができました：

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

しかし、1つの長い行は読みにくいので、分割することをお勧めします。2つのメソッド呼び出しの2行です。
さて、この行が何をしているかについて話しましょう。

先に述べたように`read_line`は、渡している文字列にユーザーが入力したものを入れますが、この場合は[`io::Result`][ioresult]<!-- ignore -->型の値も返します。Rustには標準ライブラリに`Result`という名前のいくつかの型があります。一般的な[`Result`][result]<!-- ignore -->と`io::Result`のようなサブモジュールの特定バージョンです。

[ioresult]: ../../std/io/type.Result.html
[result]: ../../std/result/enum.Result.html

`Result`型は、*列挙型*[enums]<!-- ignore -->です。列挙型は固定値のセットを持つことができる型であり、その値は列挙型の*variants*と呼ばれます。第6章では列挙型について詳しく説明します。

[enums]: ch06-00-enums.html

`Result`の場合、バリアントは`Ok`か`Err`です。`Ok`は操作が成功したことを示し、`Ok`の内部は正常に生成された値です。`Err`は操作が失敗したことを意味し、`Err`は操作の失敗の理由や理由についての情報を含んでいます。

これらの`Result`型の目的は、エラー処理情報をエンコードすることです。`Result`型の値は、どんな型と同様、それらに定義されたメソッドを持っています。`io::Result`のインスタンスには、呼び出すことができる[`expect`メソッド][expect]<!-- ignore -->があります。`io::Result`のこのインスタンスが`Err`の値なら、`expect`はプログラムがクラッシュし、`expect`に引数として渡したメッセージを表示します。`read_line`メソッドが`Err`を返した場合、それは基礎となるオペレーティングシステムからのエラーの結果である可能性があります。`io::Result`のこのインスタンスが`Ok`の値なら、`expect`は`Ok`が保持している戻り値をとり、その値だけを返します。この場合、その値はユーザーが標準入力に入力したバイト数です。

[expect]: ../../std/result/enum.Result.html#method.expect

`expect`を呼び出さなければ、プログラムはコンパイルされますが警告が表示されます。

```text
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
warning: unused `std::result::Result` which must be used
  --> src/main.rs:10:5
   |
10 |     io::stdin().read_line(&mut guess);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: #[warn(unused_must_use)] on by default
```

Rustは`read_line`から返された`Result`を使用していないことを警告します。これはプログラムがエラーを処理しなかったことを示しています。


警告を抑制する正しい方法は、実際にエラー処理を書くことですが、問題が発生したときにこのプログラムをクラッシュさせたいので、`expect`を使うことができます。エラーからの回復については、第9章で学習します。

### `println!`で値を表示するプレースホルダ

これまでに追加されたコードではさらに議論すべき行が1つあります。

```rust,ignore
println!("You guessed: {}", guess);
```

この行はユーザーの入力を保存した文字列を表示します。`{}`はプレースホルダです。値を保持する小さなカニの挟み込みを`{}`と考えます。中括弧を使用して複数の値を出力することができます。中括弧の最初のセットは、書式文字列の後にリストされた最初の値を保持し、2番目のセットは2番目の値を保持します。`println!`の1回の呼び出しで複数の値を出力すると、次のようになります。

```rust
let x = 5;
let y = 10;

println!("x = {} and y = {}", x, y);
```

このコードは`x = 5 and y = 10`を出力します。

### 最初の部分をテストする

数あてゲームの最初の部分をテストしましょう。`cargo run`を使って実行します。

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

この時点でゲームの最初の部分が完了します。キーボードから入力を受け取り、それを出力しています。

## 秘密番号の生成

次に、ユーザーが予想しようとする秘密の番号を生成する必要があります。ゲームは複数回遊ぶので、秘密の番号は毎回異なるはずです。1と100の間の乱数を使用して、ゲームがそれほど難しくないようにしましょう。Rustには標準ライブラリに乱数機能はまだ含まれていません。しかし、Rustチームは[`rand`クレート][randcrate]を提供しています。

[randcrate]: https://crates.io/crates/rand

### クレートを使用してより多くの機能を利用する

クレートはRustのソースコードファイルの集合であることに注意してください。構築してきたプロジェクトは*バイナリクレート*です。これは実行可能ファイルです。`rand` クレートは、他のプログラムで使用されることを意図したコードを含む*ライブラリクレート*です。

Cargoの外部クレートの使用は、それが本当に価値のあることです。`rand`を使用するコードを書く前に、*Cargo.toml*ファイルを修正して、`rand`クレートを依存関係として含める必要があります。そのファイルを開き、Cargoが作成した`[dependencies]`セクションのヘッダの下に次の行を追加してください。

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[dependencies]

rand = "0.3.14"
```

*Cargo.toml*ファイルではヘッダーの後に続くすべてが、別のセクションが開始するまで続くセクションの一部です。`[dependencies]`セクションは、Cargoにプロジェクトが依存している外部クレートと、必要とするクレートのバージョンを伝える場所です。この場合、`rand`クレートはセマンティックバージョニング指定子`0.3.14`で指定します。Cargoはバージョン番号を書くための標準である[セマンティックバージョニング][semver]<!-- ignore -->（時には*SemVer*と呼ばれる）を理解しています。数字`0.3.14`は実際には`^0.3.14`の省略形で、「バージョン0.3.14と互換性のある公開APIを持つすべてのバージョン」を意味します。

[semver]: http://semver.org

さて、コードを変更せずに、リスト2-2に示すようにプロジェクトをビルドしましょう。

```text
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.14
 Downloading libc v0.2.14
   Compiling libc v0.2.14
   Compiling rand v0.3.14
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
```

<span class="caption">リスト 2-2:依存関係として`rand`クレートを追加した後に`cargo build`を実行した結果
</span>

なるバージョン番号が表示されることがあります（しかし、それらはすべてSemVerのおかげでコードと互換性があります）。また、行の順序が異なる場合があります。

Cargoは外部依存関係を持つようになったので、[Crates.io][cratesio]のデータのコピーである*レジストリ*からすべての最新バージョンを取得します。Crates.ioは、Rustエコシステムの人々が他の人が使用するためのオープンソースのRustプロジェクトを投稿する場所です。

[cratesio]: https://crates.io

レジストリを更新した後、Cargoは`[dependencies]`セクションをチェックし、まだ持っていないクレートをダウンロードします。この場合、`rand`は依存関係としてリストされていましたが、`rand`は`libc`に依存しているため、Cargoは`libc`のコピーも取得しました。クレートをダウンロードした後、Rustはそれらをコンパイルし、利用可能な依存関係でプロジェクトをコンパイルします。

`cargo build`を何も変更せずに直ちに実行すると、`Finished`行以外の出力は得られません。Cargoはそれがすでに依存関係をダウンロードしてコンパイルしていることを知っており、*Cargo.toml*ファイルでそれらについて何も変更していません。Cargoは、コードを何も変更していないことも知っているので、それを再コンパイルしません。何もすることなく終了します。

*src/main.rs*ファイルを開き、簡単な変更を加えて保存してもう一度ビルドすると、2行の出力しか表示されません。

```text
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
```

これらの行は、Cargoが*src/main.rs*ファイルへの小さな変更でビルドを更新するだけであることを示しています。依存関係は変更されていないので、Cargoはすでにダウンロードしてコンパイルしたものを再利用できることを認識しています。コードの一部を再構築するだけです。

#### *Cargo.lock*ファイルによる再現可能なビルドの保証

Cargoには、あなたや他の誰かがコードをビルドするたびに、同じアーティファクトを再構築できる仕組みがあります。Cargoは別のバージョンを指定するまで指定した依存関係のバージョンのみを使用します。例えば、`rand`クレートの`v0.3.15`バージョンが来て、重要なバグ修正が含まれているだけでなく、コードを壊す回帰が含まれているとどうなりますか？

この問題への答えは、最初に`cargo build`を実行したときに作成され、*guessing_game*ディレクトリにある*Cargo.lock*ファイルです。初めてプロジェクトをビルドするとき、Cargoは基準に適合するすべてのバージョンの依存関係を把握し、*Cargo.lock*ファイルに書き込みます。将来的にプロジェクトをビルドする場合、Cargoは*Cargo.lock*ファイルが存在することを確認し、バージョンを再検討するすべての作業を行うのではなく、指定されたバージョンを使用します。これにより、自動的に再現可能なビルドが可能になります。つまり、*Cargo.lock*ファイルのおかげで明示的にアップグレードするまでプロジェクトは`0.3.14`のままです。

#### 新しいバージョンを取得するためのクレートの更新

クレートを更新したいとき、Cargoは*Cargo.lock*ファイルを無視して、*Cargo.toml*の仕様に合った最新バージョンをすべて調べる、もう一つのコマンド`update`を提供します。それが有効ならば、Cargoはこれらのバージョンを*Cargo.lock*ファイルに書き込みます。

しかし、デフォルトでは、Cargoは`0.3.0`より大きく、`0.4.0`より小さいバージョンのみを探します。`rand`クレートが`0.3.15`と`0.4.0`という2つの新しいバージョンをリリースした場合、`cargo update`を実行すると、次のように表示されます。

```text
$ cargo update
    Updating registry `https://github.com/rust-lang/crates.io-index`
    Updating rand v0.3.14 -> v0.3.15
```

この時点で、*Cargo.lock*ファイルの変更が気付くでしょう。現在使用している`rand`クレートのバージョンは`0.3.15`です。

`rand`バージョン`0.4.0`や`0.4.x`シリーズのバージョンを使いたい場合は、*Cargo.toml*ファイルを次のように更新する必要があります。

```toml
[dependencies]

rand = "0.4.0"
```

次に`cargo build`を実行すると、Cargoは利用可能なクレートレジストリを更新し、指定した新しいバージョンに従って`rand`要件を再評価します。

第14章で議論する[Cargo][doccargo]<!--ignore -->と[エコシステム][doccratesio]<!--ignore -->についてはもっと多くのことが説明されていて、そこですべてのことを知ることができます。Cargoはライブラリの再利用を非常に簡単にします。したがって、Rust開発者は多くのパッケージから組み立てられた小さなプロジェクトを書くことができます。

[doccargo]: http://doc.crates.io
[doccratesio]: http://doc.crates.io/crates-io.html

### 乱数の生成

*Cargo.toml*に`rand`クレートを追加したので、`rand`を使い始めましょう。次のステップは、リスト2-3に示すように、*src/main.rs*を更新することです。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

<span class="caption">リスト 2-3: 乱数を生成するコードを追加する</span>

まず、Rustに外部依存として`rand`クレートを使用することを知らせる行を追加します。これは`use rand`を呼び出すのと同じことですが、`rand`の前に`rand::`を置くことで`rand`クレートの何かを呼び出すことができます。

次に、`use rand::Rng`という別の`use`行を追加します。`Rng`トレイトは、乱数ジェネレータが実装するメソッドを定義しており、これらのメソッドを使用するには、このトレイトが有効でなければなりません。第10章では、トレイトの詳細について説明します。

また、中央に2行追加しています。`rand::thread_rng`関数は、使用する特定の乱数ジェネレータを提供します。現在の実行スレッドのローカルであり、オペレーティングシステムによってシードされるものです。次に、乱数ジェネレータで`gen_range`メソッドを呼び出します。 このメソッドは、`use rand::Rng`ステートメントでスコープ化した`Rng`トレイトによって定義されています。`gen_range`メソッドは引数として2つの数字をとり、それらの間に乱数を生成します。これは下限に含まれていますが上限にはありませんので、1と100の間の数値を要求するには`1`と`101`を指定する必要があります。

> 注：使用するトレイトやクレートから呼び出すメソッドや関数はわかりません。クレートを使用するための説明は各クレートのドキュメントに記載されています。Cargoのもうひとつの特徴は、`cargo doc --open`コマンドを実行することです。このコマンドは、すべての依存関係によって提供されるドキュメントをローカルにビルドし、ブラウザで開くようにします。例えば、`rand`枠内の他の機能に興味があるなら、`cargo doc --open`を実行し、左側のサイドバーで`rand`をクリックしてください。

コードに追加した2行目は秘密の番号を出力します。これは、テストできるようにプログラムを開発している間は便利ですが、最終版から削除します。プログラムが起動するとすぐに答えが表示されるのは、ゲームとしてよくありません。

数回プログラムを実行してみてください。

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4
$ cargo run
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

異なる乱数を得なければなりません。それらはすべて1から100までの数字でなければなりません。

## 予想を秘密の番号と比較する

ユーザー入力と乱数があるので、比較することができます。リスト2-4にその手順を示します。ここで説明するように、このコードはまだコンパイルされません。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    // ---snip---

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

<span class="caption">リスト 2-4: 2つの数値を比較する戻り値の処理</span>

最初の新しい点は`std::cmp::Ordering`という型を標準ライブラリのスコープに持ち込む別の`use`ステートメントです。`Result`と同様に`Ordering`も別の列挙型ですが、`Ordering`のバリアントは`Less`、`Greater`、`Equal`です。これらは、2つの値を比較したときの3つの結果です。

次に、`Ordering`型を使用する新しい5行を追加します。

`cmp`メソッドは2つの値を比較し、比較可能なもので呼び出すことができます。それは比較したいものを参照します。ここでは `guess`と`secret_number`を比較しています。次に、`use`ステートメントでスコープに入れた`Ordering`列挙型のバリアントを返します。[`match`][match]<!-- ignore -->式を使用して、`guess`の`cmp`の呼び出しから返された`Ordering`のバリアントに基づいて、次に何をするかを決定します。 

[match]: ch06-02-match.html

`match`式は*arm*で構成されています。armは*パターン*と`match`式の先頭に与えられた値がそのarmのパターンに合っていれば実行されるべきコードで構成されています。Rustは`match`に与えられた値を取り、順番に各armのパターンを調べます。`match`コンストラクトとパターンはRustの強力な機能であり、コードが遭遇する可能性のあるさまざまな状況を表現し、それらをすべて処理することを保証します。これらの機能については、それぞれ第6章と第18章で詳しく説明します。

ここで使用される`match`式で何が起こるかの例を見てみましょう。50と38を比較すると、`cmp`メソッドは`Ordering::Greater`を返します。これは、50が38より大きいためです。`match`式は`Ordering::Greater`値を取得し、各armのパターンのチェックを開始します。最初のarmのパターン、`Ordering::Less`を見て、`Ordering::Greater`の値が`Ordering::Less`と一致しないので、そのarmのコードを無視して次の腕に移動します。次のarmのパターン、`Ordering::Greater`は、`Ordering::Greater`にマッチします。そのarmの関連コードが実行され、`Too big!`を画面に表示します。このシナリオでは最後のarmを見る必要がないため、`match`式は終了します。

しかし、リスト2-4のコードはまだコンパイルされません。試してみましょう。

```text
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
error[E0308]: mismatched types
  --> src/main.rs:23:21
   |
23 |     match guess.cmp(&secret_number) {
   |                     ^^^^^^^^^^^^^^ expected struct `std::string::String`, found integral variable
   |
   = note: expected type `&std::string::String`
   = note:    found type `&{integer}`

error: aborting due to previous error
Could not compile `guessing_game`.
```

エラーのコアには、*mismatched types*と書かれています。Rustは強力な静的タイプのシステムを持っています。ただし、型推論もあります。`let mut guess = String::new();`と書いたとき、Rustは`guess`が`String`でなければならないと推論することができました。一方、`secret_number`は数値型です。いくつかの数値型は1〜100の値をとります。`i32`は32ビットの数値です。`u32`は符号なしの32ビット数です。64ビットの数値は他のものと同様で`i64`です。Rustのデフォルトは`secret_number`の型である`i32`です。Rustが異なる数値型を推論させるような型情報を他の場所に追加しない限り、そうではありません。エラーの原因は、Rustが文字列と数値型を比較できないためです。

最終的には、プログラムが読み込んだ`String`を数値型に変換して、`guess`と数値的に比較することができます。これを行うには、以下の2行を`main`関数本体に追加します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
// --snip--

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

2つの新しい行は次のとおりです。

```rust,ignore
let guess: u32 = guess.trim().parse()
    .expect("Please type a number!");
```

`guess`という名前の変数を作成します。しかし、プログラムが既に`guess`という名前の変数を持っているのではないでしょうか？すでに存在していますが、Rustは新しい値で`guess`の以前の値を*シャドー*することができます。この機能は、値をある型から別の型に変換する場合によく使用されます。シャドーイングでは、`guess`と`guess_str`のような2つの固有の変数を強制的に作成するのではなく、`guess`変数名を再利用できます（第3章では、シャドーイングについて詳しく説明しています）。

`guess`を`guess.trim().parse()`という表現にバインドします。式の`guess`は元の`guess`を参照していますが、`guess`は入力された`String`です。`String`インスタンスの`trim`メソッドは、最初と最後の空白を取り除きます。`u32`は数字だけを含むことができますが、ユーザーは<span class="keystroke">enter</span>を押して`read_line`を満たす必要があります。ユーザーが<span class="keystroke"></span>を入力すると、改行文字が文字列に追加されます。たとえば、<span class="keystroke">5</span>と入力し、<span class="keystroke">enter</span>を入力すると、`guess`は` 5\n`のようになります。`\n`は"改行"を表し、<span class="keystroke"></span>を押した結果です。`trim`メソッドは`\n`を削除し、`5`だけを返します。

Stringの[`parse`メソッド][parse]<!-- ignore -->は、文字列をある種の数値にパースします。このメソッドはさまざまな数値型をパースできるので、`let guess: u32`を使ってRustに必要な正確な数値型を伝える必要があります。`guess`の後のコロン(`:`)は、Rustに変数の型に注釈を付けるよう指示します。Rustにはいくつかの組み込みの数値型があります。ここで見られる`u32`は符号なしの32ビット整数です。これは小さな正の数のための良いデフォルトの選択です。さらに、このサンプルプログラムの`u32`アノテーションと`secret_number`との比較は、`secret_number`も`u32`でなければならないとRustが推論することを意味します。これで比較は同じ型の2つの値になります。

[parse]: ../../std/primitive.str.html#method.parse

`parse`を呼び出すと簡単にエラーが発生する可能性があります。例えば、文字列に`A👍％`が含まれていた場合、それを数値に変換する方法はありません。`parse`メソッドは失敗する可能性があるので、`read_line`メソッドと同様に`Result`型を返します（前述の「`Result`型による潜在的な障害の処理」を参照）。`expect`メソッドを使うことによって、この`Result`を同じ方法で扱います。`parse`が文字列から数値を生成できなかったので、`Result`の`Err`バリアントを返した場合、`expect`はゲームをクラッシュさせ、与えたメッセージを出力します。`parse`が文字列を数字に変換することができれば`Result`の`Ok`バリアントを返し、`expect`は`Ok`から望む数字を返します。

プログラムを実行しましょう。

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43 secs
     Running `target/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

推測の前に空白が追加されたにもかかわらず、プログラムはユーザーが76を推測したと考えました。異なる種類の入力で異なる動作を確認するためにプログラムを数回実行します。数字を正しく推測し、あまりにも低い数字を推測してください。

現在ゲームのほとんどを動作させていますが、ユーザーは1つしか推測できません。
ループを追加して変更しましょう。

## ループによる複数の推測の許可


`loop`キーワードは無限ループを作成します。これを追加して、ユーザーに番号を予測する機会を増やします。


<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
// --snip--

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
        }
    }
}
```

ご覧のとおり、推測入力プロンプトからすべてをループに移しました。ループ内の行をそれぞれ4つのスペースでインデントして、プログラムを再度実行してください。プログラムは、行った通りのことをやっているので、新しい問題があることに注意してください。別の予測を永遠に求めて、ユーザーが終了できません。

ユーザーは、キーボードショートカット<span class="keystroke">ctrl-c</span>を使用して、プログラムを停止することができます。しかし、前述したように、この状況から脱出する別の方法があります。ユーザーが非数字の答えを入力すると、プログラムがクラッシュします。ここに示すように、ユーザーは終了するためにそれを利用することができます。

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50 secs
     Running `target/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread 'main' panicked at 'Please type a number!: ParseIntError { kind: InvalidDigit }', src/libcore/result.rs:785
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/guess` (exit code: 101)
```

`quit`と打つと実際にゲームは終了しますが、他の数字以外の入力も終了します。しかし、これは最適とはいえません。正しい数字が推測されると、ゲームは自動的に停止するようにします。

### 正しい予測の後に終了する

ユーザーが`break`ステートメントを追加して勝利したときにゲームを終了するようにプログラムしましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
// --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

`You win!`の後に`break`行を追加すると、ユーザが秘密の番号を正しく予想するとプログラムがループを抜けます。ループを終了することは、ループが `main`の最後の部分であるため、プログラムを終了することも意味します。

### 無効な入力の処理

ゲームの動作をさらに洗練するために、ユーザが非数字を入力したときにプログラムをクラッシュさせるのではなく、ゲームが予測を続けることができるように数字以外の数字を無視してみましょう。`guess`が`String`から`u32`に変換される行を変更することで、これを行うことができます。

```rust,ignore
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

`expect`の呼び出しから`match`式に切り替えることでエラーを処理する方法が変わります。`parse`は`Result`型を返し、`Result`は`Ok`や`Err`型を持つ列挙型です。`cmp`メソッドの`Ordering`結果で行ったように、ここでは`match`式を使用しています。

`parse`が文字列を数値に変換することができれば、結果の数値を含む`Ok`値を返します。その`Ok`値は最初のarmのパターンと一致し、`match`式は`parse`が生成し`Ok`値の中に入れた`num`値を返します。

`parse`が文字列を数値に変換できない場合、エラーに関するより多くの情報を含む`Err`を返します。`Err`は最初の`match`armの`Ok(num)`パターンとは一致しませんが、2番目のアームの`Err(_)`パターンと一致します。アンダースコア`_`はキャッチオール値です。この例では、内部にどのような情報があっても、すべての`Err`値を一致させたいとしています。プログラムは2番目のarmのコード`continue`を実行します。これは`loop`の次の繰り返しに行き、別の予測を求めます。そのため、プログラムは`parse`が遭遇する可能性があるすべてのエラーを無視します。

これで、プログラムのすべてが期待どおりに動作するはずです。 試してみましょう。

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

1つの小さな最終調整で、我々は数あてゲームを終了します。プログラムは依然として秘密の番号を出力していることを思い出してください。それはテストのためにはうまくいきましたが、ゲームを破壊します。秘密の番号を出力する`println!`を削除しましょう。
リスト2-5に最終コードを示します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

<span class="caption">リスト 2-5: 完全な数あてゲームのコード</span>

## まとめ

この時点で、正常に数あてゲームを構築できました。

このプロジェクトは`let`、`match`、メソッド、関連機能、外部クレートの使用など、多くの新しいRustコンセプトを紹介する実践的な方法でした。次のいくつかの章では、これらの概念についてより詳しく学びます。第3章では、変数、データ型、関数などのほとんどのプログラミング言語に存在する概念と、Rustでそれらを使用する方法を示します。第4章では所有権について説明しますが、Rustは他の言語とは異なります。第5章では構造体とメソッドの構文について説明し、第6章では列挙型がどのように機能するかについて説明します。
