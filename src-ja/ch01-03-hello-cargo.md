## Hello, Cargo!

CargoはRustのビルドシステムとパッケージマネージャーです。ほとんどのRust開発者はこのツールを使ってRustプロジェクトを管理しています。Cargoはコードの構築、コードが依存するライブラリのダウンロード、それらのライブラリの構築など、多くのタスクを処理するためです（コードが必要とするライブラリを*依存関係*と呼びます）。

これまでに書いたような最も単純なRustプログラムは、依存関係を持ちません。そのため、Hello World! Cargoプロジェクトでは、コードの作成を担当する部分のみCargoを使用します。もっと複雑なRustプログラムを書くと、依存関係が追加され、Cargoを使用してプロジェクトを開始すると、依存関係を追加する方がはるかに簡単になります。

Rustプロジェクトの大部分はCargoを使用しているため、この本の残りの部分では、Cargoも使用していると仮定しています。「インストール」セクションで説明した公式のインストーラを使用した場合、CargoにはRustがインストールされます。他の方法でRustをインストールした場合、Cargoがインストールされているかどうかは、端末に次のように入力して確認してください。

```text
$ cargo --version
```

バージョン番号が表示されている場合は、インストールされています。`command not found`などのエラーが表示された場合は、インストール方法に関するドキュメントを参照して、Cargoを個別にインストールしてください。

### Cargoでプロジェクト作成

Cargoを使用して新しいプロジェクトを作成し、元のHello、world！プロジェクトとどのように違うのかを見てみましょう。*projects*ディレクトリ（またはコードを保存することに決めた場所）に戻ります。次に、すべてのオペレーティングシステムで次のコマンドを実行します。

```text
$ cargo new hello_cargo
$ cd hello_cargo
```

最初のコマンドは、*hello_cargo*という新しいディレクトリを作成します。 プロジェクト*hello_cargo*に名前を付け、Cargoは同じ名前のディレクトリにファイルを作成します。

*hello_cargo*ディレクトリに移動し、ファイルを一覧表示します。Cargoが2つのファイルと1つのディレクトリ（*Cargo.toml*ファイルと*main.rs*ファイルのある*src*ディレクトリ）を生成したことがわかります。また、*.gitignore*ファイルとともに新しいGitリポジトリを初期化しました。

> 注意：Gitは一般的なバージョン管理システムです。`cargo new`を` --vcs`フラグを使って別のバージョン管理システムを使うか、バージョン管理システムを使わないように変更することができます。`cargo new --help`を実行すると利用可能なオプションが表示されます。

選択したテキストエディタでCargo.tomlを開きます。*リスト1-2*のコードに似ているはずです。
<span class="filename">ファイル名: Cargo.toml</span>

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]
```

<span class="caption">リスト 1-2: `cargo new`によって生成された*Cargo.toml*の内容</span>

このファイルは、Cargoの設定形式である[*TOML*][toml]<!-- ignore --> （*Tom's Obvious、Minimal Language*）形式です。
[toml]: https://github.com/toml-lang/toml

最初の行`[package]`は、次の文がパッケージを構成していることを示すセクション見出しです。このファイルに情報を追加すると、他のセクションも追加されます。

次の4行は、プログラムをコンパイルするためにCargoが必要とする構成情報（名前、バージョン、およびそれを書いた人）を設定します。Cargoはお使いの環境から名前とemail情報を取得します。その情報が正しくない場合は、ここで情報を修正してからファイルを保存してください。Appendix Eの`edition`キーについてお話します。

最後の行`[dependencies]`は、プロジェクトの依存関係のリストを表示するセクションの先頭です。Rustでは、コードのパッケージは*クレート*と呼ばれます。このプロジェクトでは他のクレートは必要ありませんが、第2章の最初のプロジェクトではこの依存関係のセクションを使用します。

*src/main.rs*を開いて見てください。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargoはリスト1-1で書いたのと同じように`Hello, world!`プログラムを生成しました。これまでのプロジェクトとCargoプロジェクトの違いは、Cargoが*src*ディレクトリにコードを配置し、*Cargo.toml*構成ファイルがトップディレクトリにあることです。

Cargoはソースファイルが*src*ディレクトリ内に存在することを期待しています。トップレベルのプロジェクトディレクトリは、READMEファイル、ライセンス情報、設定ファイル、およびコードに関係のないその他のものです。Cargoを使用すると、プロジェクトを整理するのに役立ちます。

`Hello, world!`のように、Cargoを使用しないプロジェクトを開始した場合 それをCargoを使用するプロジェクトに変換することができます。プロジェクトコードを*src*ディレクトリに移動し、適切な*Cargo.toml*ファイルを作成します。

### Cargoプロジェクトのビルドと実行

さあ、Cargoで`Hello, world!`プログラムを構築して実行すると、何が違うのか見てみましょう。*hello_cargo*ディレクトリから、次のコマンドを入力してプロジェクトをビルドします。

```text
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

このコマンドは、現在のディレクトリではなく、*target/debug/hello_cargo*（Windowsでは*target\debug\hello_cargo.exe*）に実行可能ファイルを作成します。このコマンドで実行可能ファイルを実行することができます。

```text
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

すべてうまくいけば、`Hello, world!`がターミナルに出力されるはずです。`cargo build`を初めて実行すると、Cargoはトップレベルで新しい*Cargo.lock*というファイルを作成します。このファイルはプロジェクトの依存関係の正確なバージョンを追跡します。このプロジェクトは依存関係がないので、ファイルは少しだけです。手動でこのファイルを変更する必要はありません。Cargoはその内容を管理します。

`cargo build`を使ってプロジェクトをビルドし、`./target/debug/hello_cargo`で実行しましたが、`cargo run`を使ってコードをコンパイルし、実行可能なすべての実行可能ファイルを一つのコマンドで実行することもできます。

```text
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

今回は、Cargoが `hello_cargo`をコンパイルしていることを示す出力が表示されなかったことに注目してください。Cargoはファイルが変更されていないことを知ったので、バイナリを実行しました。ソースコードを変更した場合、Cargoはプロジェクトを実行する前にプロジェクトを再構築しています。この出力は次のようになります。

```text
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargoには、`Cargo check`というコマンドもあります。このコマンドは、コードがコンパイルされているかどうかをすぐにチェックしますが、実行可能ファイルは生成しません。

```text
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

なぜ実行可能ファイルを生成しないほうがよいのでしょうか？実行可能ファイルを生成するステップをスキップするので、`cargo check`は` cargo build`よりもはるかに高速です。コードを書いている間、作業を絶えずチェックしているなら、`cargo check`を使ってプロセスをスピードアップしましょう。このように、多くのRust開発者は定期的に`cargo check`を行い、コンパイルを確実にするためにプログラムを書きます。実行可能ファイルを使用する準備ができたら、`cargo build`を実行します。

これまでにCargoについて学んだことをまとめます

* `cargo build`や`cargo check`を使用してプロジェクトを構築することができます。
* `cargo run`を使ってプロジェクトをビルドして1つのステップで実行することができます。
* ビルド結果をコードと同じディレクトリに保存する代わりに、Cargoは*target/debug*ディレクトリに格納します。

Cargoを使用することのもう一つの利点は、作業しているオペレーティングシステムに関係なくコマンドが同じであることです。そのため、LinuxとmacOS、Windowsのための具体的な手順は提供しなくなります。

### リリース

プロジェクトが最終的にリリース準備が整ったら、`cargo build --release`を使って最適化してコンパイルすることができます。このコマンドは、*target/debug*の代わりに*target/release*に実行可能ファイルを作成します。最適化によってRustコードはより速く実行されますが、プログラムをコンパイルするのにかかる時間が長くなります。これは、2つの異なるプロファイルがあります。1つは開発用で迅速かつ頻繁にリビルドすることが可能で、もう1つは繰り返しリビルドされない最終的なプログラム用です。コードの実行時間をベンチマークしているならば、`cargo build --release`を実行し、*target/release*の実行ファイルでベンチマークしてください。

### 条約としてのCargo

単純なプロジェクトでは、Cargoは`rustc`を使用するだけでは価値がありませんが、プログラムが複雑になるにつれてその価値は証明されます。 複数のcratesで構成される複雑なプロジェクトでは、Cargoがビルドを調整するのがずっと簡単です。

`hello_cargo`プロジェクトはシンプルですが、Rustの残りの仕事で使う本当のツールの多くを使います。実際、既存のプロジェクトを操作するには、次のコマンドを使用してGitを使ってコードをチェックアウトし、そのプロジェクトのディレクトリに変更してビルドします。

```text
$ git clone someurl.com/someproject
$ cd someproject
$ cargo build
```
Cargoの詳細については、[ドキュメント][its documentation]を参照してください。

[its documentation]: https://doc.rust-lang.org/cargo/

## まとめ

すでにRustの素晴らしいスタートを切ることができています。この章では、以下の方法を学習しました。

* `rustup`を使って最新の安定版Rustをインストールしてください
* 新しいRustバージョンへのアップデート
* ローカルにインストールされたドキュメントを開く
* Hello world!を作成して`rustc`を直接使って実行する
* Cargoを使用して新しいプロジェクトを作成して実行する

これは、Rustのコードの読み書きに慣れるためのより充実したプログラムを構築する素晴らしい時間です。したがって、第2章では、数あてゲームプログラムを作成します。Rustで一般的なプログラミングの概念がどのように機能するかを学ぶことから始める場合は、第3章を参照してから第2章に戻ります。
