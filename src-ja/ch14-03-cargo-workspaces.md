## Cargoのワークスペース

第12章では、バイナリクレートとライブラリクレートを含むパッケージを作成しました。プロジェクトが進展するにつれて、ライブラリクレートは引き続き大きくなり、パッケージを複数のライブラリクレートにさらに分割したいと思うかもしれません。このような状況のためにCargoには、*ワークスペース*という機能があり、複数の関連パッケージを一元管理して管理することができます。

### ワークスペースを生成する

*ワークスペース*は、同じ*Cargo.lock*と出力ディレクトリを共有する一連のパッケージです。ワークスペースを使ってプロジェクトを作ってみましょう。ワークスペースの構造に集中できるように、簡単なコードを使用します。ワークスペースを構成するには複数の方法があります。1つの共通の方法を示します。バイナリと2つのライブラリを含むワークスペースがあります。主な機能を提供するバイナリは、2つのライブラリに依存します。一つのライブラリは`add_one`関数を提供し、二つ目のライブラリは`add_two`関数を提供します。これらの3つのクレートは、同じワークスペースの一部になります。作業領域の新しいディレクトリを作成します。

```text
$ mkdir add
$ cd add
```

次に、*add*ディレクトリにワークスペース全体を構成する*Cargo.toml*ファイルを作成します。このファイルには、他の*Cargo.toml*ファイルで見た`[package]`セクションやメタデータはありません。代わりに、`[workspace]`セクションから始まり、バイナリクレートへのパスを指定することでワークスペースにメンバを追加することができます。この場合、そのパスは*adder*です：

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[workspace]

members = [
    "adder",
]
```

次に、*add*ディレクトリ内で`cargo new`を実行して`adder`バイナリクレートを作成します。

```text
$ cargo new adder
     Created binary (application) `adder` project
```

この時点で、`cargo build`を実行して作業領域を構築することができます。*add*ディレクトリのファイルは次のようになります。

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

ワークスペースには、コンパイルされた成果物が配置される最上位に1つの*target*ディレクトリがあります。`adder`クレートには独自の*target*ディレクトリがありません。*adder*ディレクトリの内側から`cargo build'を実行しても、コンパイルされた成果物は*add/adder/target*ではなく*add/target*になります。Cargoは、このようなワークスペースの*target*ディレクトリを構造化します。なぜなら、ワークスペース内のクレートは互いに依存するためです。各クレートに独自の*target*ディレクトリがある場合、各クレートはワークスペース内の他の各クレートを再コンパイルして、自身の*target*ディレクトリにアーティファクトを持たなければなりません。1つの*target*ディレクトリを共有することで、不必要な再構築を避けることができます。

### ワークスペース内に2番目のクレートを作成する

次に、ワークスペースに`add-one`という名前の別のメンバークレートを作成します。*Cargo.toml*を変更して、`members`リストに*add-one*を指定します

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

次に、`add-one`という名前の新しいライブラリクレートを生成します。

```text
$ cargo new add-one --lib
     Created library `add-one` project
```

*add*ディレクトリにこれらのディレクトリとファイルがあるはずです。

```text
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

*add-one/src/lib.rs*ファイルに、`add_one`関数を追加しましょう。

<span class="filename">ファイル名: add-one/src/lib.rs</span>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

ワークスペースにライブラリクレートがあるので、バイナリクレート`adder`をライブラリクレート`add-one`に依存させることができます。まず、*addder/Cargo.toml*に`add-one`にパス依存を追加する必要があります。

<span class="filename">ファイル名: adder/Cargo.toml</span>

```toml
[dependencies]

add-one = { path = "../add-one" }
```

Cargoでは、ワークスペース内のクレートが互いに依存するとは想定されていないため、クレート間の依存関係について明示する必要があります。

次に、`adder`クレート内の`add-one`クレートから`add_one`関数を使用しましょう。*adder/src/main.rs*ファイルを開き、上に`use`行を追加して、新しい`add-one`ライブラリクレートをスコープに入れてください。リスト14-7のように、`main`関数を変更して`add_one`関数を呼び出します。

<span class="filename">ファイル名: adder/src/main.rs</span>

```rust,ignore
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}!", num, add_one::add_one(num));
}
```

<span class="caption">リスト 14-7:`add-one`ライブラリクレートを`adder`クレートから使用する</span>

最上位の*add*ディレクトリで`cargo build`を実行してワークスペースを構築しましょう。

```text
$ cargo build
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68 secs
```

*add*ディレクトリからバイナリクレートを実行するには、`-p`引数を使用し、パッケージ名を`cargo run`で指定する必要があります。

```text
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

これは*addder/src/main.rs*のコードを実行します。これは`add-one`クレートに依存します。

#### ワークスペースの外部クレートに依存する

ワークスペースには、各クレートのディレクトリに*Cargo.lock*があるのではなく、ワークスペースの最上位に*Cargo.lock*ファイルが1つしかありません。これにより、すべてのクレートがすべての依存関係の同じバージョンを使用していることが保証されます。 *adder/Cargo.toml*と*add-one/Cargo.toml*ファイルに`rand`クレートを追加すると、Cargoはそれらの両方を`rand`の一つのバージョンに解決し、それを一つの*Cargo.lock*に記録します。同じ依存関係を使用するワークスペース内のすべてのクレートを作成すると、ワークスペース内のクレートが常に互いに互換性があることを意味します。*add-one/Cargo.toml*ファイルの`[dependencies]`セクションに`rand`クレートを追加して、`add-one`クレートで`rand`クレートを使用できるようにしましょう。

<span class="filename">ファイル名: add-one/Cargo.toml</span>

```toml
[dependencies]

rand = "0.3.14"
```

これで、*add-one/src/lib.rs*ファイルに`extern crate rand;`を追加でき、`add`ディレクトリで`cargo build`を実行することでワークスペース全体をビルドすると、randクレートを持ってきてコンパイルします。

```text
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.14
   --snip--
   Compiling rand v0.3.14
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18 secs
```

*Cargo.lock*には、`add-one`の`rand`への依存性に関する情報が含まれるようになりました。しかし、`rand`がワークスペースのどこかで使用されていても、*Cargo.toml*ファイルに`rand`を追加しない限り、ワークスペースの他のクレートには使用できません。 例えば、`adder`クレートの*adder/src/main.rs*ファイルに`extern crate rand;`を追加すると、エラーが発生します。

```text
$ cargo build
   Compiling adder v0.1.0 (file:///projects/add/adder)
error: use of unstable library feature 'rand': use `rand` from crates.io (see
issue #27703)
 --> adder/src/main.rs:1:1
  |
1 | use rand;
```

これを修正するには、`adder`クレート用の*Cargo.toml*ファイルを編集し、`rand`もそのクレートの依存関係であることを示します。`adder`クレートを作ると、*Cargo.lock*の`adder`の依存関係のリストに`rand`が追加されますが、`rand`の追加コピーはダウンロードされません。 Cargoは、`rand`クレートを使用する作業スペースのすべてのクレートが同じバージョンを使用することを保証しています。同じバージョンの`rand`をワークスペース全体で使用すると、複数のコピーを持たず、ワークスペース内のクレートが互いに互換性があるため、スペースが節約されます。

#### ワークスペースにテストを追加する

もう一つの機能強化のために、`add_one::add_one`関数のテストを`add_one`クレートの中に追加しましょう。

<span class="filename">ファイル名: add-one/src/lib.rs</span>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

トップレベルの*add*ディレクトリで`cargo test`を実行してください。

```text
$ cargo test
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running target/debug/deps/add_one-f0253159197f7841

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/adder-f88af9d2cc175a5e

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

出力の最初のセクションは、`add-one`の`it_works`テストが成功したことを示しています。次のセクションでは、`adder`クレートにゼロテストが見つかり、最後のセクションで`add-one`クレートでのドキュメンテーションテストがゼロであることが示されています。このように構造化されたワークスペースで`cargo test`を実行すると、ワークスペース内のすべてのクレートのテストが実行されます。

また、`-p`フラグを使用してテストするクレートの名前を指定することで、トップレベルディレクトリからワークスペース内のある特定のクレートのテストを実行することもできます。

```text
$ cargo test -p add-one
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/add_one-b3235fea9a156f74

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

この出力は`cargo test`が`add-one`クレートのテストを実行し、`adder`クレートテストを実行しなかったことを示しています。

ワークスペース内のクレートを*https://crates.io/*に公開する場合、ワークスペース内の各クレートは個別に公開する必要があります。`cargo publish`コマンドは`--all`フラグや`-p`フラグを持たないので、各クレートのディレクトリに移動し、ワークスペースの各クレートに`cargo publish`を実行してクレートを公開する必要があります。

さらに練習するには、 `add-one`クレートと同様の方法で`add-two`クレートをこのワークスペースに追加してください。

プロジェクトが成長するにつれて、ワークスペースの使用を検討してください。一つの大きなコードよりも小さく、個々のコンポーネントを理解する方が簡単です。さらに、ワークスペース内にクレートを保持すると、頻繁に変更されると、それらの間の調整をより簡単に行うことができます。
