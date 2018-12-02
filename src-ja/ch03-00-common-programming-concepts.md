# プログラミングの共通概念


この章ではほぼすべてのプログラミング言語で表示される概念と、Rustでどのように動作するかについて説明します。多くのプログラミング言語には共通点が共通しています。この章で紹介するコンセプトは、Rust独自のものではありませんが、Rustのコンテキストで説明しこれらのコンセプトを使用するための規則を説明します。

具体的には、変数、基本タイプ、関数、コメント、および制御フローについて学習します。これらの基盤はすべてのRustプログラムに含まれています。

## キーワード

Rust言語には、他の言語と同様に、その言語でのみ使用するために予約された*キーワード*があります。変数や関数の名前としてこれらの単語を使用することはできません。ほとんどのキーワードは特別な意味を持っています。これらのキーワードを使用してRustプログラムでさまざまなタスクを実行します。それらに関連する現在の機能はありませんが、今後はRustに追加される可能性のある機能のために予約されています。Appendix Aにキーワードのリストがあります。

## 識別子

変数、関数、構造体、たくさんの概念について説明します。これらのすべてに名前が必要です。Rustの名前は「識別子」と呼ばれ、空でないASCII文字列で構成できますが、いくつかの制限があります。

* 最初の文字は文字です。
* 残りの文字は英数字または_です。

または、

* 最初の文字は_です。
* 識別子は複数の文字です。_単独では識別子ではありません。
* 残りの文字は英数字または_です。

### 生識別子

場合によっては、別の目的のためにキーワードである名前を使用する必要があるかもしれません。たぶん、あなたはCライブラリから来ている*match*という名前の関数を呼び出す必要があります。ここで'match'はキーワードではありません。これを行うには、"生識別子"を使うことができます。生の識別子は`r#`で始まります。

```rust,ignore
let r#fn = "this variable is named 'fn' even though that's a keyword";

// call a function named 'match'
r#match();
```

生識別子はあまり必要ではありませんが、時々必要になります。