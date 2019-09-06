## スライス


所有権を持たない別のデータ型は*スライス*です。スライスを使用すると、コレクション全体ではなくコレクション内の要素の連続したシーケンスを参照できます。

小さなプログラミング上の問題があります。文字列をとり、その文字列で見つかった最初の単語を返す関数を書くとします。関数が文字列内にスペースを見つけられない場合は、文字列全体が1つの単語でなければならないため、文字列全体が返されるとします。

この関数のシグニチャについて考えてみましょう。

```rust,ignore
fn first_word(s: &String) -> ?
```

この関数`first_word`は、引数として`&String`を持っています。今回は所有権を望まないのでこれは問題ありません。しかし、何を返すべきですか？実際に文字列の*部分*について話す方法を持っていません。しかし、単語の終わりのインデックスを返すことができます。リスト4-7に示すように、それを試してみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

<span class="caption">リスト 4-7: `String`引数のスペースのインデックス値を返す`first_word`関数</span>

`String`要素を要素単位で調べ、値が空白かどうかをチェックする必要があるので、`as_bytes`メソッドを使って、`String`をバイトの配列に変換します。

```rust,ignore
let bytes = s.as_bytes();
```

次に、`iter`メソッドを使用してバイト配列のイテレータを作成します。

```rust,ignore
for (i, &item) in bytes.iter().enumerate() {
```

イテレータについては第13章で詳しく説明します。今の段階では`iter`はコレクション内の各要素を返すメソッドであり、`iter`の結果を`enumerate`でラップして、各要素をタプルの一部として返すメソッドであることに注意してください。`enumerate`から返されたタプルの最初の要素はインデックスであり、2番目の要素は要素への参照です。これは、インデックスを自分で計算するよりも少し便利です。

`enumerate`メソッドはタプルを返すので、Rustの他の場所と同様に、パターンを使ってそのタプルを破棄することができます。したがって、`for`ループではタプルのインデックスに`i`を、タプルに1バイトの`&item`を持つパターンを指定します。`.iter().enumerate()`から要素への参照を取得するので、パターンには`&`を使います。

`for`ループの中で、バイトリテラル構文を使ってスペースを表すバイトを検索します。スペースを見つけるとその位置を返します。それ以外の場合は、`s.len()`を使って文字列の長さを返します。

```rust,ignore
    if item == b' ' {
        return i;
    }
}
s.len()
```

文字列の最初の単語の終わりのインデックスを見つける方法がありますが問題があります。それ自身で`usize`を返していますが、`&String`の文脈では意味のある数字です。言い換えれば、`String`とは別の値なので、それが将来有効であるという保証はありません。リスト4-7の `first_word`関数を使用するリスト4-8のプログラムを考えてみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
# fn first_word(s: &String) -> usize {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return i;
#         }
#     }
#
#     s.len()
# }
#
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // wordは値5を取得します

    s.clear(); // これは文字列を空にし""と等しくします

    // wordはここでも値5を持っていますが、意味のある値5を使用できるStringはありません。 
    // wordが完全に無効になりました！
}
```

<span class="caption">リスト 4-8: `first_word`関数を呼び出して結果を格納し`String`の内容を変更します</span>

このプログラムはエラーなしでコンパイルされ`s.clear()`を呼び出した後に`word`を使用した場合も同様です。`word`は`s`の状態に全く接続されていないので`word`は値`5`を含んでいます。この値に`s`を変数`s`で使用して最初の単語を抽出しようとしましたが、`word`に`5`を保存してから`s`の内容が変わったのでバグでしょう。

`word`のインデックスが`s`のデータと同期しなくなることを心配するのは退屈でエラーが起こりやすいです。`second_word`関数を書くと、これらのインデックスを管理することはさらに脆弱になります。そのシグニチャは次のようになります。

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

ここでは開始位置と終了位置を追跡していますが、特定の状態のデータから計算された値はさらに多くなりますが、その状態にまったく結びついていません。3つの無関係な変数を浮動させて、同期させておく必要があります。

Rustには文字列スライスというこの問題の解決策があります。

### 文字列スライス

*文字列スライス*は`String`の一部への参照であり次のようになります。

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

これは`String`全体を参照するのと似ていますが、余分な`[0..5]`ビットが付いています。`String`全体への参照ではなく`String`の一部への参照です。`start..end`構文は、`start`で始まり`end`まで続く範囲です。`end`を含めたい場合、`..`の代わりに`..=`を使うことができます。

```rust
let s = String::from("hello world");

let hello = &s[0..=4];
let world = &s[6..=10];
```

`=`は `..`と`..=`の違いは最後の数字を含めるかどうかで覚えておくと良いでしょう。

`[starting_index..ending_index]`を指定することで括弧内のスコープを使用してスライスを作成できます。ここで`beginning_index`はスライスの最初の位置で、`ending_index`はスライスの最後の位置より1つ多くなります。内部的にはスライスデータ構造はスライスの開始位置と長さを格納します。これは`ending_index`から`starting_index`を差し引いたものに相当します。したがって、`let world = &s[6..11];`の場合、`world`は`s`の7番目のバイトと5の長さの値を指すポインタを含むスライスになります。

図4-6にこれを示します。

<img alt="world containing a pointer to the 6th byte of String s and a length 5" src="img/trpl04-06.svg" class="center" style="width: 50%;" />

<span class="caption">図 4-6: `String`の一部を参照する文字列スライス</span>

Rustの`..`のスコープ構文では、最初のインデックス(ゼロ)から開始する場合は2つのピリオドの前に値をドロップできます。言い換えればこれらは等しいです。

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

同じトークンでスライスに`String`の最後のバイトが含まれている場合は、末尾の数字を削除することができます。これは、これらが等しいことを意味します。

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

両方の値をドロップして、文字列全体をスライスすることもできます。したがって、これらは等しいです。

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 注意：文字列スライス範囲のインデックスは有効なUTF-8文字境界で指定する必要があります。マルチバイト文字の途中で文字列スライスを作成しようとすると、プログラムはエラーで終了します。文字列スライスを導入する目的で、このセクションでのみASCIIを仮定しています。UTF-8処理のより詳細な説明は、第8章の「文字列」のセクションにあります。

この情報を念頭に置いて、スライスを返すために`first_word`を書き直してみましょう。"string slice"を表す型は`&str`と書かれています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

リスト4-7と同じ方法で単語の終わりのインデックスを取得します。スペースの最初のオカレンスを探します。スペースを見つけると文字列の始まりとスペースのインデックスを開始と終了のインデックスとして使用して文字列スライスを返します。

`first_word`を呼び出すと、基礎となるデータに結びついた単一の値が返されます。値はスライスの開始点への参照とスライス内の要素の数で構成されます。

スライスを返すことは`second_word`関数のためにも機能します。

```rust,ignore
fn second_word(s: &String) -> &str {
```

コンパイラは`String`への参照が有効であることを保証するので、簡単なAPIを持っています。リスト4-8のプログラムのバグを覚えておいてください。最初の単語の終わりまでインデックスを取得した後、インデックスを無効にするために文字列をクリアした場合はどうでしょうか？そのコードは論理的に間違っていましたが、即時のエラーは表示されませんでした。空文字列で最初の単語インデックスを使用しようとした場合、問題が後で表示されます。スライスはこのバグを不可能にし、コードの問題がはるかに早いことを知らせます。`first_word`のスライス版を使用すると、コンパイル時エラーが発生します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // エラー！

    println!("the first word is: {}", word);
}
```

コンパイラのエラーは次のとおりです。

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:10:5
   |
8  |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
9  | 
10 |     s.clear(); // エラー！
   |     ^^^^^^^^^ mutable borrow occurs here
11 |     
12 |     println!("the first word is: {}", word);
   |                                       ---- borrow later used here
```

借用のルールから、何かへの不変な参照があれば、変更可能な参照を取ることができないことを思い出してください。`clear`は`String`を切り捨てる必要があるため、変更可能な参照を取得しようとしますが失敗します。RustはAPIを使いやすくしただけでなく、コンパイル時にクラス全体のエラーをなくしました。

#### 文字列リテラルはスライス

バイナリの内部に格納されている文字列リテラルについて話したことを思い出してください。スライスについて知ったので、文字列リテラルを正しく理解することができます。

```rust
let s = "Hello, world!";
```

`s`の型は`&str`です。これはバイナリの特定の点を指すスライスです。これは文字列リテラルが不変である理由です。`&str`は不変の参照です。


#### 引数としての文字列スライス

リテラルと`String`のスライスを取ることができることを知っていると、`first_word`の改善点が1つ増えています。

```rust,ignore
fn first_word(s: &String) -> &str {
```

経験豊富なRust開発者は`String`と`str`の両方で同じ関数を使うことができるので代わりに次のように書くでしょう。

```rust,ignore
fn first_word(s: &str) -> &str {
```

文字列スライスがある場合はそれを直接渡すことができます。`String`を持っていれば`String`全体のスライスを渡すことができます。`String`への参照の代わりに文字列スライスを取る関数を定義することは、APIを機能を失うことなく、より一般的かつ有用なものにします。

<span class="filename">ファイル名: src/main.rs</span>

```rust
# fn first_word(s: &str) -> &str {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return &s[0..i];
#         }
#     }
#
#     &s[..]
# }
fn main() {
    let my_string = String::from("hello world");

    // first_word works on slices of `String`s
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_word works on slices of string literals
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

### その他のスライス

想像の通り、文字列スライスは文字列に固有のものです。しかし、より一般的なスライスタイプもあります。
この配列を考えてみましょう。

```rust
let a = [1, 2, 3, 4, 5];
```

文字列の一部を参照したいのと同じように、配列の一部を参照することもできます。次のようにします。

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```

このスライスは`&[i32]`型です。これは最初の要素と長さへの参照を格納することによって、文字列スライスと同じように動作します。この種のスライスをあらゆる種類の他のコレクションに使用します。これらのコレクションについては、第8章のベクトルについて説明します。

## まとめ

所有権、借用、およびスライスの概念は、コンパイル時にRustプログラムにおけるメモリの安全性を保証します。Rust言語は、他のシステムプログラミング言語と同じ方法でメモリ使用量を制御できますが、所有者が範囲外になったときにデータ所有者がデータを自動的にクリーンアップするということは、追加コードを記述したりデバッグする必要なくこの制御を得ることができます。

所有権はRustの他の部分のどれくらいが影響を受けるのかに影響を及ぼします。そのためこれらの概念については残りの部分でさらに詳しく説明します。第5章に進み`struct`でデータをまとめてみましょう。
