## `panic!`で回復不能なエラー

時として、コードに悪いことが起こることもあります。それに対して、できることは何もありません。このような場合、Rustには`panic!`マクロがあります。`panic!`マクロが実行されると、プログラムは失敗メッセージを出力し、スタックの巻き戻しとクリーンアップを行い、終了します。これは最も一般的には、ある種のバグが検出されたときに発生し、プログラマーにエラーを処理する方法が明確でない場合に発生します。

> ### パニックに対してスタックを巻き戻すか異常終了するか
>
> デフォルトでは、パニックが発生するとプログラムは*巻き戻し*を開始します。これは、Rustがスタックをさかのぼり、遭遇した各関数からデータをクリーンアップすることを意味します。しかし、このさかのぼってクリーンアップするには多くの仕事が発生します。代わりに、即座に*異常終了する*があります。これは、クリーンアップせずにプログラムを終了します。この場合、プログラムが使用していたメモリは、オペレーティングシステムによってクリーンアップする必要があります。プロジェクトでバイナリを可能な限り小さくする必要がある場合は、の*Cargo.toml*ファイルの適切な`[profile]`セクションに`panic='abort'`を追加することで、パニック時に巻き戻しから異常終了に切り替えることができます。たとえば、リリースモードでパニックを中止する場合は、次のように追加します。
> 
> ```toml
> [profile.release]
> panic = 'abort'
> ```

単純なプログラムでpanic!の呼び出しを試してみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust,should_panic,panics
fn main() {
    panic!("crash and burn");
}
```

プログラムを実行すると、次のように表示されます。

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25 secs
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2:4
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

`panic!`を呼び出すと、最後の2行に含まれるエラーメッセージが表示されます。最初の行は、パニックメッセージとパニックが発生したソースコード内の場所を示しています。*src/main.rs:2:4*は、*src/main.rs*ファイルの2行目4文字目であることを示します。

この場合、示された行はコードの一部で、その行に行くと`panic!`マクロ呼び出しが表示されます。panic!呼び出しが、自分のコードが呼び出しているコードの一部になっている可能性もあります。エラーメッセージで報告されるファイル名と行番号が、結果的に`panic!`呼び出しに導いた自分のコードの行ではなく、`panic!`マクロが呼び出されている他人のコードになるでしょう。問題の原因となっているコードの部分を理解するために、`panic!`呼び出しが出てきた関数のバックトレースを使うことができます。次に、バックトレースの詳細を次に説明します。

### `panic!`バックトレースを使用する

マクロを直接呼び出すコードではなく、コードのバグのために、ライブラリから`panic!`呼び出されたときの様子を見てみましょう。コードリスト9-1に、ベクタのインデックスで要素にアクセスしようとするコードがあります。

<span class="filename">ファイル名: src/main.rs</span>

```rust,should_panic,panics
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

<span class="caption">リスト 9-1: ベクタの境界を超えて要素へのアクセスを試み、`panic!`の呼び出しを発生させる</span>

ここでは、ベクタの100番目の要素(インデックスがゼロから始まるため、インデックス99にあります)にアクセスしようとしていますが、要素は3つしかありません。この状況では、Rustはパニックします。`[]`を使うと要素を返すことになっていますが、無効なインデックスを渡すとRustが返すことのできる要素がなくなります。

C言語など他の言語は、この場面で欲しいものではないにもかかわらず、要求したものを返そうとしてきます。例えば、ベクタの要素に対応するメモリ上の場所、たとえメモリがベクタに属していなくても、返してきます。これは*buffer overread*と呼ばれ、攻撃者が配列の後に格納されるべきではないデータを読み取るような方法でインデックスを操作することができれば、セキュリティ脆弱性につながる可能性があります。

この種の脆弱性からプログラムを保護するために、存在しないインデックスの要素を読み込もうとすると、Rustは実行を停止し続行を拒否します。それを試して見てみましょう。

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is
99', /checkout/src/liballoc/vec.rs:1555:10
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

このエラーは、自分のファイルではなくファイル*vec.rs*を指しています。これが標準ライブラリの`Vec<T>`の実装です。ベクタ`v`で`[]`を使ったときに実行されるコードは*vec.rs*の中にあり、それは`panic!`が実際に起こっている場所です。

次の注釈行は、`RUST_BACKTRACE`環境変数を設定して、エラーの原因となったもののバックトレースを取得できることを示しています。*バックトレース*は、この時点までに呼び出されたすべての関数のリストです。Rustのバックトレースは他の言語と同じように動作します。バックトレースを読むためのコツは、先頭から始め、書き込んだファイルが見えるまで読むことです。そこが問題が発生した場所です。自分のファイルに言及している行の上の行は、自分のコードが呼び出しているコードです。以下の行は自分のコードを呼び出すコードです。これらの行には、Rustの核となる標準ライブラリのコード、または使用しているクレートが含まれている場合があります。`RUST_BACKTRACE`環境変数を0以外の任意の値に設定して、バックトレースを取得します。リスト9-2と同様の出力が得られます。

```text
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', /checkout/src/liballoc/vec.rs:1555:10
stack backtrace:
   0: std::sys::imp::backtrace::tracing::imp::unwind_backtrace
             at /checkout/src/libstd/sys/unix/backtrace/tracing/gcc_s.rs:49
   1: std::sys_common::backtrace::_print
             at /checkout/src/libstd/sys_common/backtrace.rs:71
   2: std::panicking::default_hook::{{closure}}
             at /checkout/src/libstd/sys_common/backtrace.rs:60
             at /checkout/src/libstd/panicking.rs:381
   3: std::panicking::default_hook
             at /checkout/src/libstd/panicking.rs:397
   4: std::panicking::rust_panic_with_hook
             at /checkout/src/libstd/panicking.rs:611
   5: std::panicking::begin_panic
             at /checkout/src/libstd/panicking.rs:572
   6: std::panicking::begin_panic_fmt
             at /checkout/src/libstd/panicking.rs:522
   7: rust_begin_unwind
             at /checkout/src/libstd/panicking.rs:498
   8: core::panicking::panic_fmt
             at /checkout/src/libcore/panicking.rs:71
   9: core::panicking::panic_bounds_check
             at /checkout/src/libcore/panicking.rs:58
  10: <alloc::vec::Vec<T> as core::ops::index::Index<usize>>::index
             at /checkout/src/liballoc/vec.rs:1555
  11: panic::main
             at src/main.rs:4
  12: __rust_maybe_catch_panic
             at /checkout/src/libpanic_unwind/lib.rs:99
  13: std::rt::lang_start
             at /checkout/src/libstd/panicking.rs:459
             at /checkout/src/libstd/panic.rs:361
             at /checkout/src/libstd/rt.rs:61
  14: main
  15: __libc_start_main
  16: <unknown>
```

<span class="caption">リスト 9-2: `RUST_BACKTRACE`環境変数をセットした時に表示される、`panic!`呼び出しが生成するバックトレース</span>

出力がとても多いです。お使いのオペレーティングシステムとRustバージョンによって出力は異なる場合があります。この情報でバックトレースを取得するには、デバッグシンボルを有効にする必要があります。`--release`フラグを付けずに`cargo build`や`cargo run`を使うと、デフォルトでデバッグシンボルが有効になります。

リスト9-2の出力では、バックトレースの11行目が、問題を引き起こしているプロジェクトの行を指しています。*src/main.rs*の4行目です。プログラムがパニックするのを避けたいのであれば、自分の書いたファイルに言及している最初の行で指し示されている場所が調査を開始する場所です。リスト9-1では、バックトレースを使用する方法を示すためにパニックになるコードを意図的に記述したましたが、パニックを修正する方法は、3つの項目のみを含むベクタからインデックス99の要素を要求しないことです。将来、コードがパニックに陥った場合、パニックを引き起こす値とそのコードが何をすべきかをコードがどのような動作を取っているのか把握する必要があります。

また、この章の後ほど、「`panic!`するかするまいか」節で`panic!`とエラー状態を扱うのにpanic!を使うべき時と使わぬべき時に戻ってきます。次は、`Result`を使用してエラーから回復する方法を見ましょう。

