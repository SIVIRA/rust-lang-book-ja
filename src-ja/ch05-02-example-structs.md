## 構造体を使用したプログラム例

構造体をいつ使用するのかを理解するために、四角形の面積を計算するプログラムを作成してみましょう。単一の変数から始めて、代わりに構造体を使用するまでプログラムをリファクタリングします。

Cargoで*rectangles*という名前の新しいバイナリプロジェクトを作成しましょう。このプロジェクトでは、ピクセルで指定された矩形の幅と高さをとり、矩形の面積を計算します。リスト5-8に短いプログラムを示します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

<span class="caption">リスト 5-8: 個別の幅と高さの変数で指定された矩形の面積を計算する</span>

次に`cargo run`を使ってこのプログラムを実行してください。

```text
The area of the rectangle is 1500 square pixels.
```

リスト5-8はさまざまな寸法で`area`関数を呼び出すことによって四角形の面積を計算しても正しい結果が得られます。幅と高さはそれらが一緒に1つの長方形を記述しているため互いに関連しています。

このコードの問題は`area`のシグニチャです。

```rust,ignore
fn area(width: u32, height: u32) -> u32 {
```

`area`関数は1つの矩形の面積を計算することになっていますが、ここで書いた関数は2つの引数を持っています。引数は関連していますが、プログラムのどこにも表示されません。幅と高さをグループ化すると、読みやすく、管理しやすくなります。第3章の「タプル型」の項でタプルを使用してこれを行う方法の1つについて既に説明しました。

### タプルによるリファクタリング

リスト5-9にタプルを使用する別のバージョンのプログラムを示します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

<span class="caption">リスト 5-9: タプルで四角形の幅と高さを指定する</span>

タプルは構造体を少し追加しましたが今では引数を1つだけ渡しています。しかし、このバージョンはあまり明確ではありません。タプルはその要素の名前を指定しないので、タプルの部分にインデックスを付ける必要があるため計算が複雑になります。

面積計算に幅と高さを混ぜても問題はありませんが、画面上に矩形を描きたい場合は問題になります。`width`はタプルインデックス`0`で、`height`はタプルインデックス`1`です。他の誰かがこのコードで作業した場合、これを把握しておく必要があります。 これらの値を忘れたり、混乱させたり、エラーを引き起こしたりするのは簡単でしょう。コードの中でデータの意味を伝えていないからです。

### 構造体によるリファクタリング：意味の追加

構造体を使用してデータにラベルを付けることによって意味を追加します。リスト5-10に示すように、使用しているタプルをパーツの名前と名前の両方を持つデータ型に変換することができます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

<span class="caption">リスト 5-10: `Rectangle`構造体の定義</span>

ここでは、構造体を定義し、それを`Rectangle`という名前にしました。中括弧の中でフィールドを`width`と`height`として定義しました。どちらも`u32`型です。そして`main`では幅`30`、高さ`50`の特定の`Rectangle`インスタンスを作成しました。

`area`関数は`rectangle`という名前の1つの引数で定義されています。その型は構造体`Rectangle`インスタンスの不変な借用です。第4章で述べたように、構造体の所有権を取るのではなく、構造体を借りたいと考えています。このように`main`は所有権を保持しており`rect1`を使うことができます。なぜなら、関数シグニチャと関数呼び出しのどこに`&`を使用するのかという理由があります。

`area`関数は`Rectangle`インスタンスの`width`と`height`フィールドにアクセスします。`area`の関数シグニチャは、`width`と `height`フィールドを使って`Rectangle`の面積を計算することを意味します。これは、幅と高さが互いに関連していることを伝え、`0`と`1`のタプルインデックス値を使用するのではなく値に説明的な名前を付けます。これは分かりやすくするための勝利です。

### Derivable Traitで便利な機能を追加する

プログラムをデバッグしている間に`Rectangle`のインスタンスを出力してすべてのフィールドの値を見ることができればいいと思います。リスト5-11は前の章で使ったように、`println!`マクロを使って試行しています。ただし、これは動作しません。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {}", rect1);
}
```

<span class="caption">リスト 5-11: `Rectangle`インスタンスを出力しようとしています</span>

このコードを実行すると、エラーが発生します。

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

`println!`マクロは多くの種類の書式設定を行うことができ、デフォルトでは`println!`は`Display`と呼ばれる書式を使用するように指示します。これまでに見たようなプリミティブ型は、デフォルトで`Display`を実装しています。なぜなら、ユーザーに`1`やその他のプリミティブ型を表示する方法が1つしかないからです。しかし、構造体では、`println!`の形式は出力のフォーマットがより明確ではありません。なぜなら、表示の可能性が増えているからです。中括弧を印刷しますか？すべてのフィールドを表示する必要がありますか？この曖昧さのために、Rustは望むものを推測しようとせず、構造体には`Display`の実装がありません。

引き続きエラーを読んでいれば、この有益なメモを見つけることができます。

```text
`Rectangle` cannot be formatted with the default formatter; try using
`:?` instead if you are using a format string
```

`println!`マクロ呼び出しは`println!("rect1 is {:?}", rect1);`のようになります。指定子 `:?`を中括弧の中に置くと`println!`は`Debug`という出力形式を使いたいと言います。`Debug`は開発者にとって有用な方法で構造体を印刷できるようにするための特性です。コードをデバッグしている間に値を見ることができます。

この変更でコードを実行しますが、まだエラーが発生します。

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Debug` is not satisfied
```

しかし、コンパイラは役立つ情報を与えてくれます。

```text
`Rectangle` cannot be formatted using `:?`; if it is defined in your
crate, add `#[derive(Debug)]` or manually implement it
```

Rustにはデバッグ情報を表示する機能がありますが、構造体にその機能を利用できるように明示的にオプトインする必要があります。これを行うためにリスト5-12に示すように、構造体定義の直前に注釈`#[derive(Debug)]`を追加します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
}
```

<span class="caption">リスト 5-12: アノテーションを追加して`Debug`トレイトを導出し、`Rectangle`インスタンスをデバッグ形式で出力します</span>

ここでプログラムを実行すると、エラーは発生せず次の出力が表示されます。

```text
rect1 is Rectangle { width: 30, height: 50 }
```

これはもっともきれいな出力ではありませんが、このインスタンスのすべてのフィールドの値を表示します。これはデバッグ時には間違いなく役立ちます。より大きな構造体がある場合、読みやすいように出力するのが便利です。そのような場合、`println!`文字列に`{:?}`の代わりに`{:#?}`を使うことができます。この例で`{:#?}`スタイルを使用すると、出力は次のようになります。

```text
rect1 is Rectangle {
    width: 30,
    height: 50
}
```

Rustはカスタムタイプに有用な振る舞いを加えることができるアノテーションを「派生」するために、いくつかの特性を提供しました。これらの特性とその振る舞いについては、Appendix C「Derivable Traits」を参照してください。これらの特性をカスタム動作で実装する方法と、第10章でトレイトの特性を作成する方法について説明します。

`area`関数は四角形の面積だけを計算する非常に特殊なものです。他の型では動作しないので、この動作を`Rectangle`構造体にもっと密接に結びつけることは役に立ちます。`area`関数を`Rectangle`型で定義された`area`*メソッド*に変換することでこのコードをどのようにリファクタリングすることができるかを見てみましょう。
