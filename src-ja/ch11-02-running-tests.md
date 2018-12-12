## テストの実行され方を制御する

`cargo run`がコードをコンパイルしてバイナリを実行するのと同様に、`cargo test`はコードをテストモードでコンパイルし、結果のテストバイナリを実行します。コマンドラインオプションを指定して、`cargo test`のデフォルト動作を変更することができます。たとえば、`cargo test`によって生成されるバイナリのデフォルトの動作は、すべてのテストを並列に実行し、テスト実行中に生成された出力をキャプチャして出力が表示されルノを防ぎ、テスト結果に関連する出力を読みやすくすることです 。

いくつかのコマンドラインオプションは`cargo test`に行き、いくつかはテストバイナリに行きます。これらの2つのタイプの引数を分けるために、`cargo test`に続いてセパレータ`--`をつけた引数と、テストバイナリに行くものをリストします。 `cargo test --help`は`cargo test`で使うオプションを表示し、`cargo test --help`はセパレータ`--`の後に使うオプションを表示します。

### テストを並列または連続して実行する

複数のテストを実行すると、デフォルトではスレッドを使用して並列実行されます。これは、テストがより速く実行されることを意味し、コードが機能しているかどうかを迅速にフィードバックできます。テストは同時に実行されているので、現在の作業ディレクトリや環境変数などの共有環境を含め、テストがお互いに依存するか、共有状態に依存しないことを確認してください。

たとえば、それぞれのテストで、*test-output.txt*という名前のディスク上にファイルを作成し、そのファイルにデータを書き込むコードを実行するとします。各テストでは、そのファイルのデータが読み込まれ、ファイルにはテストごとに異なる特定の値が含まれていることが示されます。テストは同時に実行されるため、あるテストで別のテストがファイルを書き込んだり読み込んだりするまでにファイルを上書きすることがあります。2番目のテストは失敗します。これは、コードが正しくないためではなく、テストが並行して実行されている間に互いに干渉したためです。1つの解決策は、各テストが異なるファイルに書き込むことを確認することです。別の解決策は、一度に1つずつテストを実行することです。

テストを並行して実行したくない場合や、使用するスレッド数をより細かく制御したい場合は、`--test-threads`フラグと使用したいスレッド数をテストバイナリに送ることができます。次の例を見てください。

```text
$ cargo test -- --test-threads=1
```

テストスレッドの数を`1`に設定し、プログラムに並列性を使用しないように指示します。1つのスレッドを使用してテストを実行すると、それらを並列に実行するより時間がかかりますが、テストが状態を共有する場合、テストは互いに干渉しません。

### 関数の出力を表示する

デフォルトでは、テストに合格すると、Rustのテストライブラリは標準出力に出力されたものをすべて取得します。たとえば、テストで `println!`を呼び出してテストに合格すると、端末に`println!`という出力は表示されません。テストが成功したことを示す行だけが表示されます。テストが失敗した場合は、残りの失敗メッセージと共に標準出力に表示されたものがすべて表示されます。

たとえば、コードリスト11-10には、パラメータの値を出力して10を返すばかげた関数と、渡されたテストと失敗したテストがあります。

<span class="filename">ファイル名: src/lib.rs</span>

```rust,panics
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

<span class="caption">リスト 11-10: `println!`を呼び出す関数用のテスト</span>

`cargo test`でこれらのテストを実行すると、次の出力が表示されます。

```text
running 2 tests
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

failures:

---- tests::this_test_will_fail stdout ----
        I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

この出力のどこにも、私が値4を得ていることがわかります。これは、渡されたテストが実行されたときに表示されるものです。 その出力がキャプチャされました。失敗したテストからの出力である`I got the value 8`はテストサマリー出力エリアに出現し、ここには、テスト失敗の原因も表示されています。

テストを渡すための印刷された値も見たい場合は、`--nocapture`フラグを使って出力キャプチャの動作を無効にすることができます。

```text
$ cargo test -- --nocapture
```

リスト11-10のテストを `--nocapture`フラグで再実行すると、次のような出力が表示されます。

```text
running 2 tests
I got the value 4
I got the value 8
test tests::this_test_will_pass ... ok
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
test tests::this_test_will_fail ... FAILED

failures:

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

テスト用の出力とテスト結果はインターリーブされていることに注意してください。その理由は、前のセクションで説明したように、テストが並行して実行されているためです。`--test-threads=1`オプションと`--nocapture`フラグを使い、出力がどのようなものか見てみましょう。

### 名前でテストの一部を実行する

場合によっては、完全なテストスイートを実行するのに時間がかかることがあります。特定の領域のコードで作業している場合は、そのコードに関連するテストのみを実行することができます。`cargo test`に引数として実行したいテストの名前を渡すことで、実行するテストを選択することができます。

テストのサブセットを実行する方法を示すために、リスト11-11に示すように、`add_two`関数の3つのテストを作成し、実行するものを選択します。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

<span class="caption">リスト 11-11: 3つの異なる名前の3つのテスト</span>

前に見たように、引数を渡さずにテストを実行すると、すべてのテストが並行して実行されます。

```text
running 3 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

#### 単独のテストを走らせる

テスト関数の名前を`cargo test`に渡して、そのテストだけを実行することができます。

```text
$ cargo test one_hundred
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out
```

`one_hundred`という名前のテストだけが実行されました。他の2つのテストはその名前と一致しません。テスト出力では、サマリー行の最後に`2 filtered out`と表示して、このコマンドが実行したものよりも多くのテストがあることがわかります。

このように複数のテストの名前を指定することはできません。`cargo test`に与えられた最初の値のみが使用されます。しかし、複数のテストを実行する方法があります。

#### 複数のテストを実行するためのフィルタリング

テスト名の一部を指定することができ、その名前と一致する名前のテストが実行されます。たとえば、テストの名前のうち2つに`add`が含まれているので、`cargo test add`を実行して2つのテストを実行できます。

```text
$ cargo test add
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 2 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

このコマンドは、名前に`add`を付けてすべてのテストを実行し、`one_hundred`という名前のテストを除外しました。また、テストが表示されるモジュールはテストの名前の一部になりますので、モジュールの名前をフィルタリングしてモジュール内のすべてのテストを実行できます。

### 特に希望のない限りテストを無視する

場合によっては、いくつかの特定のテストが実行に非常に時間がかかることがあるので、ほとんどの`cargo test`の実行中にそれらを除外したいかもしれません。実行したいすべてのテストを引数としてリストするのではなく、時間のかかるテストに `ignore`属性を使って注釈を付けて除外することができます。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

`#[test]`の後に、除外したいテストに`#[ignore]`行を追加します。テストを実行すると、`it_works`が実行されますが、`expensive_test`は実行しません。

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out
```

`expensive_test`関数は`ignored`としてリストされます。無視されたテストだけを実行したい場合は、`cargo test -- --ignored`を使うことができます。

```text
$ cargo test -- --ignored
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

どのテストを実行するかを制御することで、`cargo test`の結果が速くなることを確認できます。`ignored`テストの結果を確認することが道理に合い、結果を待つだけの時間ができたときに、 代わりに`cargo test -- --ignored`を走らせることができます。