## テストの体系化

この章の冒頭で述べたように、テストは複雑な規律であり、二兎により用語や体系化が異なります。Rustコミュニティでは、テストについて、*単体テスト*と*結合テスト*の2つのカテゴリに分かれて考えています。単体テストは、小さくて集中的で、一度に1つのモジュールを単独でテストし、プライベートインターフェイスをテストできます。統合テストは完全にあなたのライブラリの外部にあり、パブリックインターフェイスのみを使用し、テストごとに複数のモジュールを実行する可能性がある他の外部コードと同じ方法でコードを使用します。

どちらのテストを書くのも、ライブラリの一部が個別かつ共同でしてほしいことをしていることを確認するのに重要なのです。

### 単体テスト

単体テストの目的は、コードの各単位を他のコードと孤立してテストし、コードがどこにあるかを素早く特定し、期待どおりに動作しないようにすることです。それぞれのファイルの* src *ディレクトリにある単体テストをテストしているコードに置きます。慣例は、各ファイルに`tests`という名前のモジュールを作成し、テスト関数を含み、`cfg(test)`でモジュールに注釈を付けることです。

#### テストモジュールと`#[cfg(test)]`

テストモジュールの `#[cfg(test)]`アノテーションは`cargo build`を実行したときではなく`cargo test`を実行したときにのみテストコードをコンパイルして実行するようRustに指示します。これにより、ライブラリが構築され、結果としてコンパイルされた成果物にスペースが節約されるだけで、テストが含まれないため、コンパイル時間が節約されます。統合テストは別のディレクトリにあるので、`#[cfg(test)]`アノテーションは必要ありません。しかし、単体テストはコードと同じファイルに入っているので、`#[cfg(test)]`を使ってそれらをコンパイル結果に含めないように指定します。

この章の最初の節で新しい`adder`プロジェクトを生成した時に、Cargoがこのコードも生成してくれたことを思い出してください:

<span class="filename">ファイル名: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

このコードは自動的に生成されたテストモジュールです。属性`cfg`は*configuration*の略であり、以下の項目は特定の設定オプションを指定した場合にのみ含めるべきであることをRustに伝えます。この場合、構成オプションはtestをコンパイルして実行するためにRustによって提供される`test`です。`cfg`属性を使うことで、`cargo test`で積極的にテストを実行する場合にのみ、Cargoはテストコードをコンパイルします。これには、`#[test]`で注釈を付けられた関数に加えて、このモジュール内にあるヘルパー関数が含まれます。

#### 非公開関数をテストする

プライベート機能を直接テストする必要があるかどうかについては、テストコミュニティ内で議論されており、他の言語ではプライベート機能をテストすることが困難または不可能になっています。どのテストイデオロギーを遵守しているかに関係なく、Rustのプライバシールールではプライベート機能をテストできます。リスト11-12のプライベート関数`internal_adder`を含むコードを考えてください。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}

pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

<span class="caption">リスト 11-12: プライベート関数のテスト</span>

`internal_adder`関数は`pub`としてマークされていませんが、テストは単にRustコードであり、`tests`モジュールはちょうど別のモジュールなので、テストのスコープに`internal_adder`を持って呼び出すことができます。プライベート関数がテストされるべきだと思っていないなら、そうするように強制するRustには何もありません。

### 結合テスト

Rustでは、統合テストは完全にライブラリの外部にあります。他のコードと同じ方法でライブラリを使用します。つまり、ライブラリの公開APIの一部である関数のみを呼び出すことができます。その目的は、ライブラリの多くの部分が正しく連携しているかどうかをテストすることです。独自に正しく動作するコード単位では、統合されたときに問題が発生する可能性があるため、統合コードのテストカバレッジも重要です。統合テストを作成するには、まず*tests*ディレクトリが必要です。

#### *tests*ディレクトリ

*src*の横のプロジェクトディレクトリの最上位に*tests*ディレクトリを作成します。Cargoはこのディレクトリに統合テストファイルを探すことを知っています。次に、このディレクトリにいくつでもテストファイルを作成することができ、Cargoはそれぞれのファイルを個々のクレートとしてコンパイルします。

統合テストを作成しましょう。リスト11-12のコードを*src/lib.rs*ファイルに残して*tests*ディレクトリを作成し、*tests/integration_test.rs*という名前の新しいファイルを作成し、リスト11-13のコードを入力します。

<span class="filename">ファイル名: tests/integration_test.rs</span>

```rust,ignore
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

<span class="caption">リスト 11-13: `adder`クレートの関数の結合テスト</span>

コードの先頭に`use adder`を追加しました。単体テストでは不要でした。なぜなら、`tests`ディレクトリの各テストは別々のクレートなので、私たちのライブラリを各テストクレートのスコープに持っていく必要があるからです。

*tests/integration_test.rs*のコードに`#[cfg(test)]`を使って注釈を付ける必要はありません。Cargoは`tests`ディレクトリを特別に扱い、`cargo test`を実行するときにのみこのディレクトリのファイルをコンパイルします。今すぐ`cargo test`を実行してください。

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running target/debug/deps/adder-abcabcabc

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-ce99bcc2479f4607

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

出力の3つのセクションには、ユニットテスト、統合テスト、およびドキュメントテストが含まれます。ユニットテストの最初のセクションは、これまでのユニットテストの1行（リスト11-12で追加した`internal`という名前）とユニットテストのサマリー行です。

統合テストのセクションは `Running target/debug/deps/integration_test-ce99bcc2479f4607`という行から始まります（出力の最後のハッシュは異なります）。次に、その統合テストでは各テスト関数の行があり、`Doc-tests adder`セクションが始まる直前に統合テストの結果の要約行があります。

より多くの単体テスト機能を追加することでユニットテストセクションに結果ラインが追加されるのと同様に、より多くのテスト機能を統合テストファイルに追加することで、この統合テストファイルのセクションに多くの結果ラインが追加されます。 各統合テストファイルには独自のセクションがあるので、*tests*ディレクトリにさらにファイルを追加すると、より多くの統合テストセクションが作成されます。

テスト関数の名前を`cargo test`の引数として指定することで、特定の統合テスト関数を実行することができます。特定の統合テストファイルですべてのテストを実行するには、`cargo test`の`--test`引数の後ろにファイル名を続けてください。

```text
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

このコマンドは、*tests/integration_test.rs*ファイル内のテストのみを実行します。

#### 結合テスト内のサブモジュール

より多くの統合テストを追加すると、*tests*ディレクトリに複数のファイルを作成して整理するのに役立つ場合があります。 たとえば、テスト機能をテストしている機能でグループ化できます。先に述べたように、*tests*ディレクトリ内の各ファイルは、別個のクレートとしてコンパイルされます。

各結合テストファイルをそれ自身のクレートとして扱うと、エンドユーザが読者のクレートを使用するかのような個別のスコープを生成するのに役立ちます。ですが、これは*tests*ディレクトリのファイルが、コードをモジュールとファイルに分ける方法に関して第7章で学んだように、*src*のファイルとは同じ振る舞いを共有しないことを意味します。

*tests*ディレクトリ内のファイルのさまざまな動作は、複数の統合テストファイルで役立つヘルパー関数ができ、第7章の「モジュールを別のファイルに移動する」節の手順に従って共通モジュールに抽出しようとした時に最も気付きやすくなります。例えば、*tests/common.rs*を作成し、そこに`setup`という名前の関数を配置したら、 複数のテストファイルの複数のテスト関数から呼び出したい`setup`に何らかのコードを追加することができます。

<span class="filename">ファイル名: tests/common.rs</span>

```rust
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

テストを再実行すると、*common.rs*ファイルのテスト出力に新しいセクションが表示されます。ただし、このファイルにはテスト関数が含まれておらず、どこからでも`setup`関数を呼び出すことはありませんでした。

```text
running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/common-b8b07b6f1be2db70

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-d993c68b431d39df

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

テスト結果に`common 0`が表示されている状態で`running 0 tests`が表示されているのは、望んでいる結果ではありません。 他の統合テストファイルといくつかのコードを共有したかっただけです。

*tests/common.rs*を作成するのではなく、テスト出力に`common`が現れるのを避けるため、*tests/common/mod.rs*を作成します。これは、Rustも理解している別の命名規則です。この方法でファイルを命名すると、Rustは`common`モジュールを統合テストファイルとして扱わないように指示します。`setup`関数コードを*tests/common/mod.rs*に移動して*tests/common.rs*ファイルを削除すると、テスト出力のセクションは表示されなくなります。*tests*ディレクトリのサブディレクトリにあるファイルは、別々のファイルとしてコンパイルされたり、テスト出力にセクションがありません。

*tests/common/mod.rs*を作成したら、統合テストファイルのいずれかからモジュールとして使用できます。*tests/integration_test.rs*の`it_adds_two`テストから`setup`関数を呼び出す例を示します。

<span class="filename">ファイル名: tests/integration_test.rs</span>

```rust,ignore
use adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

`mod common;`宣言はリスト7-25で示したモジュール宣言と同じです。次に、テスト関数で`common::setup()`関数を呼び出すことができます。

#### バイナリクレート用の結合テスト

プロジェクトが*src/main.rs*ファイルのみを含み、*src/lib.rs*ファイルを持たないバイナリクレートである場合、*tests*ディレクトリに統合テストを作成して関数を呼び出すことはできません *src/main.rs*ファイルで`use`ステートメントでスコープに定義されています。ライブラリークレートのみが、他のクレートが使用できる機能を公開します。バイナリクレートは、単独で実行されることを意図しています。

これは、バイナリを提供するRustプロジェクトが*src/lib.rs*ファイルに存在するロジックを呼び出す簡単な*src/main.rs*ファイルを持っている理由の1つです。その構造を使用して、統合テストは、重要な機能を利用できるようにするために、ライブラリクレートを`use`でテストできます。重要な機能が動作する場合は、*src/main.rs*ファイル内の少量のコードも同様に動作し、少量のコードをテストする必要はありません。

## まとめ

Rustのテスト機能は、たとえ変更を加えたとしても、コードがどのように機能して期待通りに機能するかを指定する方法を提供します。単体テストはライブラリの別々の部分を個別に実行し、プライベートな実装の詳細をテストできます。統合テストでは、ライブラリの多くの部分が正しく連携していることを確認し、ライブラリのパブリックAPIを使用して、外部コードが使用するのと同じ方法でコードをテストします。Rustのタイプのシステムと所有権のルールはいくつかの種類のバグを防ぐのに役立ちますが、テストがロジックバグを減らすためには、コードがどのように動作することが予想されるかに関係しています。

この章で学んだ知識とこれまでの章で学んだ知識を組み合わせてプロジェクトを進めましょう。