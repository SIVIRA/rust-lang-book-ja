## 参照と借用

コードリスト4-5のタプルコードの問題は、`String`を呼び出し関数に返さなければならないということです。なぜなら、`calculate_length`の呼び出しの後に`String`を使用できるようにするためです。

値の所有権を取得するのではなく、オブジェクトへの参照を引数として持つ`calculate_length`関数を定義して使用する方法は次のとおりです。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

最初に変数宣言内のすべてのタプルコードと関数の戻り値がなくなったことに注目してください。次に`&s1`を`calculate_length`に渡す定義において、`String`ではなく`&String`にしていることに注目してください。

これらのアンパサンドは*参照*であり、あなたはそれを所有することなく何らかの値を参照することができます。図4-5にダイアグラムを示します。

<img alt="`String s1`を指す`&String s`" src="img/trpl04-05.svg" class="center" />

<span class="caption">図 4-5: `String s1`を指す`&String s`の図</span>

> 注：`&`を使って参照することの逆は*逆参照*であり、逆参照演算子`*`で行われます。第8章では逆参照演算子をいくつか使用し、第15章では逆参照の詳細について説明します。

ここで関数呼び出しを詳しく見てみましょう

```rust
# fn calculate_length(s: &String) -> usize {
#     s.len()
# }
let s1 = String::from("hello");

let len = calculate_length(&s1);
```

`&s1`は`s1`の値を*参照*していますが、それを所有していない参照を作成させます。所有していないので、スコープ外になっても、その参照元が指している値は削除されません。

同様に関数のシグニチャは`&`を使用して、引数`s`の型が参照であることを示します。説明のコメントを追加しましょう。

```rust
fn calculate_length(s: &String) -> usize { // sは文字列への参照
    s.len()
} // ここで、sはスコープ外になる。しかし、それは所有権を持っていないため
  // 何も起こりません。
```

変数`s`が有効なスコープは、関数の引数のスコープと同じですが、所有権を持たないためスコープから外れるときに参照が指すものを削除しません。関数が実際の値ではなく引数として参照を持つ場合、所有権を持たないため、所有権を返すために値を返す必要はありません。

これを関数引数の*借用*として参照を持つといいます。実際の生活の場合と同様に、人が何かを所有していればその人からそれを借りることができます。使い終わったらそれを返さなければなりません。

借りているものを変更しようとするとどうなりますか？リスト4-6のコードを試してみてください。それは動作しません。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

<span class="caption">リスト 4-6: 借用した値を変更しようとする</span>

次のようなエラーになります。

```text
error[E0596]: cannot borrow immutable borrowed content `*some_string` as mutable
 --> error.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- use `&mut String` here to make mutable
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ cannot borrow as mutable
```

デフォルトで変数が不変であるのと同じように参照もそうです。参照しているものを変更することはできません。

### 変更可能な参照

リスト4-6のコードのエラーを少し修正ことで実現できます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

まず、`s`を`mut`に変更しなければなりませんでした。次に`&mut s`で可変参照を作成し`some_string: &mut String`で可変参照を受け入れる必要があります。

しかし、変更可能な参照には大きな制限があります。特定のスコープ内の特定のデータに対して、1つの変更可能な参照のみを持つことができます。そのため、次のコードはエラーになります。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

次のようなエラーになります。

```text
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:10
  |
4 | let r1 = &mut s;
  |          ------ first mutable borrow occurs here
5 | let r2 = &mut s;
  |          ^^^^^^ second mutable borrow occurs here
6 | println!("{}, {}", r1, r2);
  |                    -- borrow later used here
```

この制限は変化を可能にしますが、非常に制御された方法です。新しいRust開発者が苦労しているのは、ほとんどの言語が好きなときにいつでも変更できるからです。

この制限を受ける利点は、Rustがコンパイル時にデータ競合を防止できることです。*データ競合*は競合状態と似ており、次の3つの動作が発生すると生じます。

* 2つ以上のポインタが同じデータに同時にアクセスします
* 少なくとも1つのポインタがデータの書き込みに使用されています
* データへのアクセスを同期するためのメカニズムはありません

データ競合は未定義の動作を引き起こし、実行時にそれらを追跡しようとしているときに診断して修正するのが難しい場合があります。Rustは、データ競合でコードをコンパイルしないのでこの問題が起こらないようにします。

中括弧を使用して新しいスコープを作成することで、*同時*でなければ複数の変更可能な参照を許可します。

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1はここで範囲外になるので、問題なく新しい参照を作成できます。

let r2 = &mut s;
```

可変と不変の参照を組み合わせるための同様の規則が存在します。このコードはエラーになります。

```rust,ignore,does_not_compile
let mut s = String::from("hello");

let r1 = &s; // 問題なし
let r2 = &s; // 問題なし
let r3 = &mut s; // 大きな問題

println!("{}, {}, and {}", r1, r2, r3);
```

次のようなエラーになります。

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:10
  |
4 | let r1 = &s; // 問題なし
  |          -- immutable borrow occurs here
5 | let r2 = &s; // 問題なし
6 | let r3 = &mut s; // 大きな問題
  |          ^^^^^^ mutable borrow occurs here
7 | 
8 | println!("{}, {}, and {}", r1, r2, r3);
  |                            -- borrow later used here
```

また、不変のものがある間*も*参照を変更することはできません。不変参照のユーザーは値が突然外から変更されることを期待しません。しかし、複数の不変参照は大丈夫です。データを読み込んでいる誰も他の人のデータの読み書きに影響を与えることはできないからです。

これらのエラーは時には苛立つかもしれませんが、Rustコンパイラは潜在的なバグを早期に（実行時ではなくコンパイル時に）発見し、問題のある場所を正確に示しています。そのためデータが思っていたものではない理由を追跡する必要はありません。

### ダングリング リファレンス

ポインタを持つ言語では、メモリへのポインタを保持しながらメモリを解放することで、他の人に与えられた可能性のあるメモリ内の位置を参照するポインタ「*ダングリング ポインタ*」を簡単に誤って作成できてしまいます。対照的にRustでは、コンパイラはダングリング リファレンスが決して参照されないことを保証します。データへの参照がある場合、コンパイラはデータへの参照が範囲外にならないようにします。

Rustがコンパイル時のエラーで防ぐことのできない、手間のかかる参照を作成しようとしましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

次のようなエラーになります。

```text
error[E0106]: missing lifetime specifier
 --> dangle.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is
  no value for it to be borrowed from
  = help: consider giving it a 'static lifetime
```

このエラーメッセージは、*ライフタイム*というまだ説明していない機能を示しています。第10章ではライフタイムについて詳しく説明します。しかし、ライフタイムについての部分を無視すると、メッセージにはなぜこのコードが問題であるかの鍵が含まれています。

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from.
```

`dangle`コードの各段階で何が起こっているのかを詳しく見てみましょう。

```rust,ignore
fn dangle() -> &String { // dangleはStringへの参照を返します

    let s = String::from("hello"); // sは新しいString

    &s // String sへの参照を返します。
} // ここでは範囲外になり削除されます。メモリから消える
  // 危険！
```

`dangle`の中に`s`が作成されているので、`dangle`のコードが終了すると、`s`は割り当て解除されます。しかし、それへの参照を返そうとしました。これは、この参照が無効な`String`を指していることを意味します。Rustはこれをさせません。

ここでの解決策は`String`を直接返すことです。

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

これは問題なく動作します。所有権が移動され、割り当てが解除されないからです。

### 参照ルール

参照について論じた内容をまとめましょう。

* 任意の時点で、1つの変更可能参照または任意の数の不変参照を持つことができます。
* 参照は常に有効でなければなりません。

次にスライスという異なる種類の参照を見てみましょう。
