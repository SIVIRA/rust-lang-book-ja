## `cargo install`でCrates.ioからバイナリをインストールする

`cargo install`コマンドを使うとバイナリクレートをローカルにインストールして使うことができます。これはシステムパッケージを置き換えるものではありません。Rustの開発者が他の人が[crates.io](https://crates.io)<!--ignore-->で共有しているツールをインストールするのに便利な方法です。 バイナリターゲットを持つパッケージのみインストールできることに注意してください。*バイナリターゲット*とは、 クレートが*src/main.rs*ファイルやバイナリとして指定された他のファイルを持つ場合に生成される実行可能なプログラムのことであり、単独では実行不可能なものの、他のプログラムに含むのには適しているライブラリターゲットとは一線を画します。通常、クレートには、*README*ファイルに、クレートがライブラリかバイナリターゲットか、両方をもつかという情報があります。

`cargo install`でインストールされたすべてのバイナリは、インストールルートの*bin*フォルダに保存されます。*rustup.rs*を使用してRustをインストールした場合、カスタム構成がない場合、このディレクトリは*$HOME/.cargo/bin*になります。`$PATH`ディレクトリに`cargo install`でインストールしたプログラムを実行できることを確認してください。

たとえば、第12章では、ファイルを検索するための`ripgrep`というgrepツールのRust実装があると述べました。`ripgrep`をインストールしたい場合は、以下を実行できます。

```text
$ cargo install ripgrep
Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading ripgrep v0.3.2
 --snip--
   Compiling ripgrep v0.3.2
    Finished release [optimized + debuginfo] target(s) in 97.91 secs
  Installing ~/.cargo/bin/rg
```

出力の最後の行には、インストールされているバイナリの場所と名前が表示されます。これは`ripgrep`の場合は`rg`です。インストールディレクトリが`$PATH`にある限り、`rg --help`を実行して、より速いRustらしいファイル検索ツールを使うことができます。
