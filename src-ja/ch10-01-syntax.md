## ジェネリックなデータ型

関数シグネチャや構造体などの項目の定義を作成するためにジェネリックスを使用できます。これらの定義は、さまざまな具体的なデータ型で使用できます。まず、ジェネリックスを使って関数、構造体、列挙型、およびメソッドを定義する方法を見ていきましょう。次に、ジェネリックスがコードのパフォーマンスにどのように影響するかについて説明します。

### 関数定義において

ジェネリックスを使用する関数を定義するとき、ジェネリックスを関数のシグネチャに置きます。ここでは、通常、パラメータと戻り値のデータ型を指定します。これにより、コードの柔軟性が高まり、コードの重複を防ぎながら、関数の呼び出し側に多くの機能が提供されます。

`largest`関数を続けます。リスト10-4はどちらもスライスから最大値を探す2つの関数を示しています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
#    assert_eq!(result, 'y');
}
```

<span class="caption">リスト 10-4: 名前とシグニチャの型のみが異なる2つの関数</span>

`largest_i32`関数は、リスト10-3で抽出したもので、スライス内で最大の`i32`を見つけます。`largest_char`関数はスライス内で最大の`char`を見つけます。関数本体には同じコードがありますので、単一の関数にジェネリック型パラメータを導入して重複を排除しましょう。

定義する新しい関数の型をパラメータ化するには、関数への値パラメータの場合と同様に、型パラメータの名前を付ける必要があります。任意の識別子を型パラメータ名として使用できますが、`T`を使用します。"慣習的にRustのパラメータ名は短く(しばしば単なる文字)であり、Rustのタイプ命名規則はキャメルケースだからです。`T`は"type"の略で、ほとんどのRustプログラマのデフォルトの選択です。

関数の本体でパラメータを使用するときは、コンパイラがその名前の意味を知るように、シグネチャにパラメータ名を宣言する必要があります。同様に、関数シグネチャに型パラメータ名を使用する場合は、型パラメータ名を宣言してから使用する必要があります。ジェネリックな`largest`関数を定義するには、型名宣言を山カッコ(`<>`)で囲み、関数名と引数リストの間に配置してください。

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

この定義を次のように読むことができます。`largest`関数は、なんらかの型`T`に関してジェネリックです。この関数は`list`という名前の1つのパラメータを持ちます。これは`T`型の値のスライスです。`largest`関数は、同じ型の`T`の値を返します。

リスト10-5は、そのシグネチャにジェネリックなデータ型を使用して、`largest`関数定義を組み合わせたものを示しています。リストには、関数を`i32`値のスライスまたは`char`値のいずれかで呼び出す方法も示されています。このコードはまだコンパイルされませんが、この章の後半で修正します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

<span class="caption">リスト 10-5: ジェネリックな型引数を使用するものの、まだコンパイルできないlargest関数の定義</span>

このコードをコンパイルすると、このエラーが発生します。

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:12
  |
5 |         if item > largest {
  |            ^^^^^^^^^^^^^^
  |
  = note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

注釈には`std::cmp::PartialOrd`があります。これは*トレイト*です。トレイトについては次のセクションで説明します。今のところ、このエラーは`T`が可能なすべての可能な型に対して`largest`の本体が機能しないことを述べています。本体の`T`型の値を比較したいので、値を並べ替えることのできる型しか使用できません。比較を可能にするために、標準ライブラリには型に対して実装できる`std::cmp::PartialOrd`特性があります（この特性の詳細については付録Cを参照してください）。"Tトレイト境界"セクションでジェネリック型が特定の特性を持つように指定する方法を学びますが、まずジェネリック型パラメータを使用する他の方法を調べてみましょう。

### 構造体定義において

`<>`構文を使用して、1つ以上のフィールドでジェネリック型パラメータを使用するように構造体を定義することもできます。コードリスト10-6は、任意の型の`x`と`y`座標値を保持する`Point<T>`構造体を定義する方法を示しています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

<span class="caption">Listing 10-6: 型`T`の`x`と`y`値を保持する`Point<T>`構造体</span>

構造体定義でジェネリックを使用する構文は、関数定義で使用される構文に似ています。まず、構造体の名前の直後に山形括弧で囲まれた型パラメータの名前を宣言します。次に、具体的なデータ型を指定する場合は、構造体定義のジェネリック型を使用できます。

`Point <T>`を定義するためにジェネリック型を1つしか使用していないので、`Point<T>`構造体はある種の`T`型よりも一般的で、`x`と`y`はどちらの型でも同じ型です。リスト10-7のように、異なる型の値を持つ`Point<T>`のインスタンスを作成すると、コードはコンパイルされません。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listing 10-7: `x`と`y`のフィールドは、両方とも同じジェネリックデータ型`T`を持つので、同じ型でなければならない</span>

この例では、整数値5を`x`に代入すると、ジェネリック型`T`が`Point <T>`のこのインスタンスの整数になることをコンパイラに知らせます。次に`x`と同じ型を持つように定義した`y`に4.0を指定すると、次のような型不一致エラーが発生します。

```text
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integral variable, found
floating-point variable
  |
  = note: expected type `{integer}`
             found type `{float}`
```

`x`と`y`が両方ともジェネリックであるが、異なるタイプを持つことができる`Point`構造体を定義するために、複数のジェネリック型パラメータを使うことができます。例えば、リスト10-8では`T`型と`U`型に対して`Point`の定義を変更することができます。`x`型は`T`型で`y`型は`U`型です。

<span class="filename">ファイル名: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listing 10-8: `Point<T, U>`は、`x`と`y`は異なる型の値になり得る</span>

`Point`のすべてのインスタンスが許可されました。定義には、必要な数のジェネリック型パラメーターを使用できますが、いくつかを使用すると、コードを読みづらくなります。多くのジェネリック型が必要な場合は、コードがより小さな部分に再構成する必要があることのサインかもしれません。

### enum定義において

構造体で行ったように、さまざまな種類の汎用データ型を保持する列挙型を定義できます。 第6章で使用した、標準ライブラリが提供する `Option <T>` enumをもう一度見てみましょう。

```rust
enum Option<T> {
    Some(T),
    None,
}
```

この定義は道理が通っているはずです。`Option<T>`は`T`型のジェネリックな列挙型で、`T`型の1つの値を保持する`Some`と値を何も保持しない`None`の2つのバリアントを持ちます。`Option<T>`列挙型を使うことで、オプショナルな値を持つという抽象的な概念を表現することができます。`Option<T>`はジェネリックスなので、オプショナルな値の型が何であってもこの抽象化を使用できます。

Enumは複数のジェネリック型も使用できます。第9章で使用した`Result`列挙体の定義は一例です。

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result`列挙型は`T`と`E`の2つの型に関してジェネリックスであり、`T`型の値を保持する`Ok`と`T`型の値を保持する`Err`の2種類のバリアントがあります。この定義は、成功する可能性のある操作（何らかの型の`T`型の値を返す）または失敗する型式（`E`型のエラーを返す）がある箇所で、`Result`列挙型を使うのが便利です。実際には、これはリスト9-3のファイルを開くために使用したものです。ファイルを開くのに成功した時に`T`に型`std::fs::File`が入り、ファイルを開く際に問題があった時に`E`に型`std::io::Error`が入ります。

自分のコード内で保持している値の型のみが異なる構造体やenum定義の場面を認識したら、代わりにジェネリックな型を使用することで重複を避けることができます。

### メソッド定義において

第5章で行ったように構造体と列挙型のメソッドを実装することができます。リスト10-9は、リスト10-6で定義した`Point<T>`構造体に`x`というメソッドを実装したものです。

<span class="filename">ファイル名: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<span class="caption">リスト 10-9: 型`T`の`x`フィールドへの参照を返す`x`というメソッドを`Point<T>`構造体に実装する</span>

ここでは、フィールド`x`のデータへの参照を返す`Point<T>`に`x`という名前のメソッドを定義しました。

`impl`の直後に`T`を宣言しなければならないので、`Point<T>`型のメソッドを実装するよう指定する必要があります。`impl`の後に`T`をジェネリック型として宣言することにより、Rustは`Point`の山括弧の型が具体的な型ではなくジェネリック型であることを識別できます。

たとえば、ジェネリック型の`Point<T>`インスタンスではなく、`Point<f32>`インスタンスでのみメソッドを実装することができます。リスト10-10では、具体的な型`f32`を使用します。つまり、`impl`の後に型を宣言しません。

```rust
# struct Point<T> {
#     x: T,
#     y: T,
# }
#
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

<span class="caption">リスト 10-10: ジェネリックな型引数`T`に対して特定の具体的な型がある構造体にのみ適用される`impl`ブロック</span>

このコードは、Point<f32>にはdistance_from_originというメソッドが存在するが、 Tがf32ではないPoint<T>の他のインスタンスにはこのメソッドが定義されないことを意味します。このメソッドは、ポイントが座標(0.0、0.0)のポイントからどれだけ離れているかを測定し、浮動小数点型に対してのみ使用可能な数学演算を使用します。

構造体定義のジェネリックな型引数は、必ずしもその構造体のメソッドシグニチャで使用するものと同じにはなりません。例えば、リスト10-11は、リスト10-8の`Point<T, U>`にメソッドmixupを定義しています。このメソッドは、他の`Point`を引数として取り、この引数は`mixup`を呼び出している`self`の`Point`とは異なる型の可能性があります。このメソッドは、型`T`の`self`の`Point`の`x`値と渡した型`W`の`Point`の`y`値から新しい`Point`インスタンスを生成します。

<span class="filename">ファイル名: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

<span class="caption">リスト 10-11: 構造体定義とは異なるジェネリックな型を使用するメソッド</span>

`main`では`x`（値が5）の`i32`と`y`の`f64`（値は`10.4`）を持つ`Point`を定義しました。`p2`変数は、`x`（値 "Hello"）を持つ文字列スライスと`y`（`c`を持つ）の`char`を持つ`Point`構造体です。引数`p2`で`p1`に`mixup`を呼び出すと、`p3`が得られ、`x`は`i32`になります。`x`は`p1`からきたからです。`p3`変数は、`y`に`char`になります。`y`は`p2`由来だからです。`println!`マクロの呼び出しは、 `p3.x = 5, p3.y = c`と出力するでしょう。

この例の目的は、いくつかの一般的なパラメータが`impl`で宣言され、いくつかがメソッド定義で宣言される状況を示すことです。 ここで、ジェネリックパラメータ`T`と`U`は、`impl`の後に宣言されています。ジェネリックパラメータ`V`と`W`は`fn mixup`の後で宣言されています。なぜならそれらはメソッドにしか関係ないからです。

### ジェネリクスを使用したコードのパフォーマンス

ジェネリック型パラメータを使用しているときにランタイムコストがあるかどうか疑問に思うかもしれません。良いニュースとして、コンパイラがジェネリクスを具体的な型がある時よりもジェネリックな型を使用したコードを実行するのが遅くならないように実装しています。

Rustは、コンパイル時にジェネリックを使用しているコードの単相化を実行することでこれを実現します。*単相化(monomorphization)*は、コンパイル時に使用される具体的な型を埋め込むことによって、ジェネリックなコードを特定のコードに変換するプロセスです。

このプロセスでは、コンパイラはコードリスト10-5のジェネリック関数を作成するために使用した手順の逆を行います。コンパイラはジェネリックコードが呼び出されるすべての場所を調べ、ジェネリックコードが呼び出される具体的な型のコードを生成します。

標準ライブラリの`Option<T>`列挙型を使った例を見てみましょう。

```rust
let integer = Some(5);
let float = Some(5.0);
```

Rustがこのコードをコンパイルすると、単相化が実行されます。その過程で、コンパイラは`Option<T>`インスタンスで使用された値を読み込み、2種類の`Option<T>`を識別します。一つは`i32`で、もう一つは`f64`です。このように、`Option<T>`の一般的な定義を`Option_i32`と`Option_f64`に展開することで、一般的な定義を特定のものに置き換えます。

コードの単形化バージョンは以下のようになります。一般的な`Option<T>`は、コンパイラによって作成された特定の定義に置き換えられます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Rustはジェネリックコードを各インスタンスの型を指定するコードにコンパイルするので、ジェネリックの使用にはランタイムコストはかかりません。コードが実行されると、各定義を手作業で複製した場合と同じように機能します。単相化のプロセスにより、 Rustのジェネリクスは実行時に非常に効率的になるのです。