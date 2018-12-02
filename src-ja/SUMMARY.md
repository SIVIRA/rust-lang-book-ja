# プログラミング言語 Rust

[前書き](foreword.md)
[導入](ch00-00-introduction.md)

## 入門

- [入門](ch01-00-getting-started.md)
    - [インストール](ch01-01-installation.md)
    - [Hello, World!](ch01-02-hello-world.md)
    - [Hello, Cargo!](ch01-03-hello-cargo.md)

- [数あてゲーム](ch02-00-guessing-game-tutorial.md)

- [プログラミングの共通概念](ch03-00-common-programming-concepts.md)
    - [変数と変更可能性](ch03-01-variables-and-mutability.md)
    - [データ型](ch03-02-data-types.md)
    - [関数の動作](ch03-03-how-functions-work.md)
    - [コメント](ch03-04-comments.md)
    - [制御フロー](ch03-05-control-flow.md)

- [所有権について](ch04-00-understanding-ownership.md)
    - [所有権とはなにか？](ch04-01-what-is-ownership.md)
    - [参照と借用](ch04-02-references-and-borrowing.md)
    - [スライス](ch04-03-slices.md)

- [構造体を使用して関係のあるデータを構造化する](ch05-00-structs.md)
    - [構造体を定義し、インスタンス化する](ch05-01-defining-structs.md)
    - [構造体を使用したプログラム例](ch05-02-example-structs.md)
    - [メソッド記法](ch05-03-method-syntax.md)

- [Enumとパターンマッチング](ch06-00-enums.md)
    - [Enumとパターンマッチング](ch06-01-defining-an-enum.md)
    - [`match`制御フロー演算子](ch06-02-match.md)
    - [`if let`による簡潔な制御フロー](ch06-03-if-let.md)

## Rustの基礎知識

- [パッケージ、Crates、モジュール](ch07-00-packages-crates-and-modules.md)
    - [ライブラリと実行可能ファイルを作成するためのパッケージとCrates](ch07-01-packages-and-crates-for-making-libraries-and-executables.md)
    - [スコープとプライバシーを制御するモジュールシステム](ch07-02-modules-and-use-to-control-scope-and-privacy.md)

- [共通コレクション](ch08-00-common-collections.md)
    - [ベクタで一連の値を保持する](ch08-01-vectors.md)
    - [UTF-8でエンコードされた文字列を格納する](ch08-02-strings.md)
    - [関連付けられた値を持つキーをハッシュマップに格納する](ch08-03-hash-maps.md)

- [エラー処理](ch09-00-error-handling.md)
    - [`panic!`で回復不能なエラー](ch09-01-unrecoverable-errors-with-panic.md)
    - [`Result`で回復可能なエラー](ch09-02-recoverable-errors-with-result.md)
    - [`panic!`すべきかするまいか](ch09-03-to-panic-or-not-to-panic.md)

- [Generic Types, Traits, and Lifetimes](ch10-00-generics.md)
    - [Generic Data Types](ch10-01-syntax.md)
    - [Traits: Defining Shared Behavior](ch10-02-traits.md)
    - [Validating References with Lifetimes](ch10-03-lifetime-syntax.md)

- [Testing](ch11-00-testing.md)
    - [Writing tests](ch11-01-writing-tests.md)
    - [Running tests](ch11-02-running-tests.md)
    - [Test Organization](ch11-03-test-organization.md)

- [An I/O Project: Building a Command Line Program](ch12-00-an-io-project.md)
    - [Accepting Command Line Arguments](ch12-01-accepting-command-line-arguments.md)
    - [Reading a File](ch12-02-reading-a-file.md)
    - [Refactoring to Improve Modularity and Error Handling](ch12-03-improving-error-handling-and-modularity.md)
    - [Developing the Library’s Functionality with Test Driven Development](ch12-04-testing-the-librarys-functionality.md)
    - [Working with Environment Variables](ch12-05-working-with-environment-variables.md)
    - [Writing Error Messages to Standard Error Instead of Standard Output](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Thinking in Rust

- [Functional Language Features: Iterators and Closures](ch13-00-functional-features.md)
    - [Closures: Anonymous Functions that Can Capture Their Environment](ch13-01-closures.md)
    - [Processing a Series of Items with Iterators](ch13-02-iterators.md)
    - [Improving Our I/O Project](ch13-03-improving-our-io-project.md)
    - [Comparing Performance: Loops vs. Iterators](ch13-04-performance.md)

- [More about Cargo and Crates.io](ch14-00-more-about-cargo.md)
    - [Customizing Builds with Release Profiles](ch14-01-release-profiles.md)
    - [Publishing a Crate to Crates.io](ch14-02-publishing-to-crates-io.md)
    - [Cargo Workspaces](ch14-03-cargo-workspaces.md)
    - [Installing Binaries from Crates.io with `cargo install`](ch14-04-installing-binaries.md)
    - [Extending Cargo with Custom Commands](ch14-05-extending-cargo.md)

- [Smart Pointers](ch15-00-smart-pointers.md)
    - [`Box<T>` Points to Data on the Heap and Has a Known Size](ch15-01-box.md)
    - [The `Deref` Trait Allows Access to the Data Through a Reference](ch15-02-deref.md)
    - [The `Drop` Trait Runs Code on Cleanup](ch15-03-drop.md)
    - [`Rc<T>`, the Reference Counted Smart Pointer](ch15-04-rc.md)
    - [`RefCell<T>` and the Interior Mutability Pattern](ch15-05-interior-mutability.md)
    - [Creating Reference Cycles and Leaking Memory is Safe](ch15-06-reference-cycles.md)

- [Fearless Concurrency](ch16-00-concurrency.md)
    - [Threads](ch16-01-threads.md)
    - [Message Passing](ch16-02-message-passing.md)
    - [Shared State](ch16-03-shared-state.md)
    - [Extensible Concurrency: `Sync` and `Send`](ch16-04-extensible-concurrency-sync-and-send.md)

- [Object Oriented Programming Features of Rust](ch17-00-oop.md)
    - [Characteristics of Object-Oriented Languages](ch17-01-what-is-oo.md)
    - [Using Trait Objects that Allow for Values of Different Types](ch17-02-trait-objects.md)
    - [Implementing an Object-Oriented Design Pattern](ch17-03-oo-design-patterns.md)

## Advanced Topics

- [Patterns Match the Structure of Values](ch18-00-patterns.md)
    - [All the Places Patterns May be Used](ch18-01-all-the-places-for-patterns.md)
    - [Refutability: Whether a Pattern Might Fail to Match](ch18-02-refutability.md)
    - [All the Pattern Syntax](ch18-03-pattern-syntax.md)

- [Advanced Features](ch19-00-advanced-features.md)
    - [Unsafe Rust](ch19-01-unsafe-rust.md)
    - [Advanced Lifetimes](ch19-02-advanced-lifetimes.md)
    - [Advanced Traits](ch19-03-advanced-traits.md)
    - [Advanced Types](ch19-04-advanced-types.md)
    - [Advanced Functions & Closures](ch19-05-advanced-functions-and-closures.md)
    - [Macros](ch19-06-macros.md)

- [Final Project: Building a Multithreaded Web Server](ch20-00-final-project-a-web-server.md)
    - [A Single Threaded Web Server](ch20-01-single-threaded.md)
    - [Turning our Single Threaded Server into a Multithreaded Server](ch20-02-multithreaded.md)
    - [Graceful Shutdown and Cleanup](ch20-03-graceful-shutdown-and-cleanup.md)

- [Appendix](appendix-00.md)
    - [A - Keywords](appendix-01-keywords.md)
    - [B - Operators and Symbols](appendix-02-operators.md)
    - [C - Derivable Traits](appendix-03-derivable-traits.md)
    - [D - Useful Development Tools](appendix-04-useful-development-tools.md)
    - [E - Editions](appendix-05-editions.md)
    - [F - Translations](appendix-06-translation.md)
    - [G - How Rust is Made and “Nightly Rust”](appendix-07-nightly-rust.md)
