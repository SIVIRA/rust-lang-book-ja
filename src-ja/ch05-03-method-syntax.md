## メソッド記法

*メソッド*は関数と似ています。`fn`キーワードとその名前で宣言されています。引数と戻り値を持つことができ、どこか他のところから呼び出されたときに実行されるコードが含まれています。しかし、メソッドは構造体（または第6章と第17章で説明する列挙型または構造体オブジェクト）のコンテキスト内で定義されている点で関数と異なり、最初の引数は常に`self`でメソッドが呼び出されている構造体のインスタンスを表します。

### メソッドの定義

リスト5-13に示すように`Rectangle`インスタンスを引数とする`area`関数を変更し、代わりに`Rectangle`構造体に定義された`area`メソッドを作ってみましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

<span class="caption">リスト 5-13: `Rectangle`構造体に` area`メソッドを定義する</span>

`Rectangle`のコンテキスト内で関数を定義するために、`impl`（実装）ブロックを開始します。次に`impl`中括弧内の`area`関数を移動させ、最初の（そしてこの場合は唯一の）引数をシグニチャ内とボディ内のどこでも`self`に変更します。`area`関数を呼び出し、`rect1`を引数として渡した`main`では*メソッド構文*を使用して、`Rectangle`インスタンスの`area`メソッドを呼び出すことができます。メソッド構文は、インスタンスの後に続きます。メソッド名、括弧、および引数の後にドットを追加します。

`area`のシグニチャでは`rectangle: &Rectangle`の代わりに`&self`を使います。Rustはこのメソッドが`impl Rectangle`コンテキストの中にあるため、`self`の型が`Rectangle`であることを知っています。`&Rectangle`のように`self`の前に`&`を使用する必要があることに注意してください。メソッドは`self`の所有権を取ることができ、他の引数と同じように`self`をここで使用しています。

関数バージョンで`&Rectangle`を使用したのと同じ理由で`&self`を選択しました。所有権を奪いたくはなく、単に構造体のデータを読み込むだけで、書き込むことはありません。メソッドを呼び出すインスタンスをメソッドの一部として変更したい場合は、最初の引数として `&mut self`を使用します。最初の引数として`self`だけを使用してインスタンスの所有権を取得するメソッドを持つことはまれです。このメソッドは、メソッドが`self`を何か他のものに変換し、呼び出し元が変換後に元のインスタンスを使用しないようにしたい場合に使用されます。

メソッド構文の使用に加えて、関数の代わりにメソッドを使う主な利点は、すべてのメソッドのシグニチャで`self`の型を繰り返す必要がなく、体系化のためです。提供するライブラリのさまざまな場所で`Rectangle`の機能をコード検索の将来のユーザーにさせるのではなく、1つの`impl`ブロックに型のインスタンスを使ってできることをすべて入れました。

> ### `->`演算子はどこですか？
>
> CとC++では、メソッドを呼び出すために2つの異なる演算子が使用されます。オブジェクトのメソッドを直接呼び出す場合は`.`を、オブジェクトへのポインタでメソッドを呼び出す場合は`->`を使います。つまり`object`がポインタの場合、`object->something()`は`(*object).something()`のようになります。
>
> Rustには`->`演算子と同等のものはありません。代わりに、Rustには*自動参照と参照解除*という機能があります。メソッドを呼び出すことは、この動作を持つRustの数少ない場所の1つです。
>
> `object.something()`でメソッドを呼び出すと、Rustは`&`、`&mut`、`*`を自動的に追加するので、`object`はメソッドのシグニチャと一致します。言い換えれば、以下は同じです。
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 最初のものははるかにきれいに見えます。この自動参照動作はメソッドが明確な受け取り`self`型を持っているために機能します。レシーバとメソッドの名前が与えられると、メソッドはメソッドが読み込み(`&self`)、変更する(`&mut self`)、または消費する(`self`)かどうかを決定することができます。Rustがメソッド受信者に暗黙的に借り入れるという事実は、実際に人間工学に基づいて所有することの大きな部分です。


### より多くの引数を持つメソッド

`Rectangle`構造体に2つ目のメソッドを実装して、メソッドを使って練習しましょう。今回は`Rectangle`のインスタンスが`Rectangle`の別のインスタンスを取得し、`Rect`が`self`の中に完全に収まる場合は`true`を返します。それ以外の場合は`false`を返します。つまり、`can_hold`メソッドを定義したら、リスト5-14に示すプログラムを書くことができます。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

<span class="caption"> リスト 5-14: 未だに書かれていない `can_hold`メソッドの使用</span>

`rect2`の両方の寸法が`rect1`の寸法よりも小さいので、`rect3`は`rect1`よりも広いので、期待される出力は次のようになります：

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

メソッドを定義したいので、`impl Rectangle`ブロックの中にあります。メソッド名は`can_hold`になり、別の`Rectangle`を引数として不変借用をとります。`rect1.can_hold(&rect2)`は`rect2`への不変借用である`&rect2`を渡します。これは`Rectangle`のインスタンスです。これは意味があります。なぜなら、`rect2`を読む必要があるからです（書き込みではなく、変更可能な借用が必要なことを意味します）。`main`が`rect2`の所有権を保持して呼び出し後に再び使用できるようにします`can_hold`メソッドです。 `can_hold`の戻り値はブール値になり、実装は`self`の幅と高さが両方とも`Rectangle`の幅と高さよりも大きいかどうかをチェックします。リスト5-15に示すリスト5-13の`impl`ブロックに新しい`can_hold`メソッドを追加しましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<span class="caption">リスト 5-15: 別の`Rectangle`インスタンスを引数として取る`Rectangle`に`can_hold`メソッドを実装する</span>

リスト5-14の`main`関数でこのコードを実行すると、希望の出力が得られます。メソッドは、`self`引数の後でシグニチャに追加する複数の引数を取ることができ、それらの引数は関数の引数と同様に機能します。

### 関連関数

`impl`ブロックのもう一つの便利な機能は`self`を引数と*しない*`impl`ブロック内の関数を定義することが許されていることです。これらは構造体に関連付けられているため*関連関数*と呼ばれます。それらは関数であり、メソッドではありません。構造体のインスタンスがないからです。既に`String::from`関連関数を使用しています。

関連関数は構造体の新しいインスタンスを返すコンストラクタでよく使用されます。たとえば、一次元の引数を1つ持ち、幅と高さの両方として使用する関連関数を定義すると、同じ値を2回指定するのではなく、正方形の`Rectangle`を作成しやすくなります。

<span class="filename">ファイル名: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

この関連する関数を呼び出すには、構造体名に`::`構文を使用します。`let sq = Rectangle::square(3);`は例です。この関数は構造体によって名前空間を持ちます。`::`構文はモジュールによって作成された関連関数と名前空間の両方に使われます。モジュールについては第7章で説明します。

### 複数の`impl`ブロック

各構造体は複数の`impl`ブロックを持つことができます。例えば、コードリスト5-15はコードリスト5-16に示すコードと同じ意味です。これはそれぞれのメソッドが独自の`impl`ブロックにあります。

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<span class="caption">リスト 5-16: 複数の`impl`ブロックを使ってリスト5-15を書き直す</span>

これらのメソッドを複数の`impl`ブロックに分割する理由はありませんが、これは有効な構文です。第10章では複数の`impl`ブロックが有用な場合を説明します。ここではジェネリック型と特性について説明します。

## まとめ

構造体を使用すると、ドメインにとって意味のあるカスタムタイプを作成できます。構造体を使用することにより、関連するデータを相互に接続し、各部分に名前を付けてコードを明確にすることができます。メソッドを使用すると、構造体のインスタンスが持つ動作を指定できます。また、関連関数を使用すると、インスタンスを使用せずに構造体に固有の名前空間機能を使用できます。

しかし、構造体はカスタムタイプを作成する唯一の方法ではありません。ツールボックスに別のツールを追加するために、Rustの列挙型に切り替えましょう。