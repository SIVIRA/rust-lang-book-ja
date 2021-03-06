## 列挙型とパターンマッチング

この例ではコードで表現したいと思う状況を見て列挙型が構造体よりも有用で適切な理由を見てみましょう。IPアドレスで作業する必要があるとします。現在、IPアドレスにはバージョン4とバージョン6の2つの主要な標準が使用されています。これらは、プログラムが遭遇するIPアドレスのための唯一の可能性です。列挙がその名前を取得するところのすべての可能な値を*列挙*できます。

どのIPアドレスもバージョン4またはバージョン6のいずれかのアドレスにすることができますが、同時に両方を使用することはできません。列挙型の値は変種の1つに過ぎないため、IPアドレスのそのプロパティは列挙型のデータ構造を適切にします。バージョン4とバージョン6の両方のアドレスは基本的にはIPアドレスなので、コードがあらゆる種類のIPアドレスに適用される状況を処理する場合、同じタイプとして扱う必要があります。

この概念をコードで表現するには、`IpAddrKind`列挙を定義し、IPアドレスが`V4`と`V6`で可能な種類を列挙します。これらは列挙型の*変種*として知られています。

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

`IpAddrKind`は、コードのどこかで使用できるカスタムデータ型になりました。

### 列挙型の値

次のように`IpAddrKind`の2つの変種のそれぞれのインスタンスを作成することができます。

```rust
# enum IpAddrKind {
#     V4,
#     V6,
# }
#
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

列挙型のバリアントはその識別子の下に名前空間があり、二重のコロンを使用してそれらを区切ることに注意してください。これが便利な理由は、`IpAddrKind::V4`と`IpAddrKind::V6`の両方の値が同じタイプである、つまり`IpAddrKind`です。たとえば`IpAddrKind`をとる関数を定義することができます。

```rust
# enum IpAddrKind {
#     V4,
#     V6,
# }
#
fn route(ip_type: IpAddrKind) { }
```

そして、この関数をどちらかのバリアントで呼び出すことができます。

```rust
# enum IpAddrKind {
#     V4,
#     V6,
# }
#
# fn route(ip_type: IpAddrKind) { }
#
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

列挙型を使用するとさらに利点があります。実際のIPアドレス*データ*を格納する方法がない瞬間、IPアドレスタイプについてもっと考えてみてください。どんな種類であるかを知っているだけです。第5章で構造体について学んだことを考えると、リスト6-1に示すように、この問題に取り組むことができます。

```rust
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

<span class="caption">リスト 6-1: `struct`を使ってIPアドレスのデータと`IpAddrKind`バリアントを保存する</span>

ここでは`IpAddrKind`型（これまでに定義した列挙型）の`kind`フィールドと`String`型の`address`フィールドの2つのフィールドを持つ`struct IpAddr`を定義しました。この構造体には2つのインスタンスがあります。最初の`home`は、関連するアドレスデータが`127.0.0.1`で`kind`が`IpAddrKind::V4`であるという値を持っています。2番目のインスタンス`loopback`は`kind`値として`IpAddrKind`のバリアント`V6`を持ち、`::1`というアドレスを持っています。構造体を使って`kind`と`address`の値を束ねていますので、このバリアントは値に関連付けられます。

構造体内の列挙型ではなく単に列挙型を使用して、各列挙型バリアントに直接データを入れて、同じ概念をより簡潔な方法で表現できます。`IpAddr`列挙型のこの新しい定義では、`V4`と`V6`の両方のバリエーションに`String`値が関連付けられていると言われています。

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

列挙型の各変種にデータを直接添付するので、余分な構造体は必要ありません。

構造体ではなく列挙型を使用することには別の利点があります。それぞれのバリアントは、関連するデータのタイプと量を異ならせることができます。バージョン4のIPアドレスは常に4つの数値コンポーネントを持ち、0から255までの値を持ちます。もし`V4`アドレスを4つの`u8`値として保存したいが`V6`アドレスを`String`値として表現したいのであれば構造体ではできません。列挙型は簡単にこのケースを処理できます。

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```
バージョン4とバージョン6のIPアドレスを格納するためにデータ構造を定義するいくつかの異なる方法を示しました。しかし、IPアドレスを保存して、どのような種類のものをエンコードしたいのかは、[標準ライブラリには使用できる定義があります][IpAddr]<!-- ignore --> と共通しています。標準ライブラリが`IpAddr`をどのように定義しているかを見ていきましょう。定義し使用した正確な列挙型と変種を持っていますが、アドレスデータを2つの異なる構造体の形でバリアント内に埋め込みます。

[IpAddr]: ../../std/net/enum.IpAddr.html

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

このコードは列挙型の内部に任意の種類のデータ（文字列、数値型、または構造体など）を入れることができることを示しています。別の列挙型を含めることもできます。

標準ライブラリに`IpAddr`の定義が含まれていても、標準ライブラリの定義をスコープに持たないので、独自の定義を作成して使用することができます。第7章では型をスコープ内に持ち込む方法について詳しく説明します。

リスト6-2の列挙型の別の例を見てみましょう。これはそのバリエーションに埋め込まれた多種多様な型を持っています。

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

<span class="caption">リスト 6-2: `Message`列挙体でそれぞれ異なる量と値の型を格納します。</span>

この列挙型にはさまざまな種類の4つのバリアントがあります。

* `Quit`には、それに関連するデータは全くありません
* `Move`はその内部に無名の構造体を含みます
* `Write`は単一の`String`を含みます
* `ChangeColor`は3つの`i32`値を含みます

リスト6-2のような変形を持つ列挙型を定義することは列挙型が`struct`キーワードを使用せず、すべてのバリアントが`Message`型の下で一緒にグループ化される点を除いて、異なる種類の構造体を定義することと似ています。次の構造体は前の列挙型が保持するデータと同じデータを保持できます。

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

しかし、それぞれ独自の型を持つ異なる構造体を使用した場合、リスト6-2で定義されている`Message`列挙型でできるように、これらのメッセージのいずれかを取る関数を一つの型で簡単に定義することはできませんでした。

列挙型と構造体の間にはもう1つの類似点があります。つまり`impl`を使って構造体にメソッドを定義できるのと同様に、enumでメソッドを定義することもできます。`call`という名前のメソッドがあり、`Message`列挙型で定義することができます。

```rust
# enum Message {
#     Quit,
#     Move { x: i32, y: i32 },
#     Write(String),
#     ChangeColor(i32, i32, i32),
# }
#
impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

メソッドの本体はメソッドを呼び出した値を取得するために`self`を使います。この例では値`Message::Write(String::from("hello"))`を持つ変数`m`を作成しました。これは`self`が`m.call()`が実行されたときに`call`メソッドを呼び出します。

標準ライブラリの別の列挙型をよく見てみましょう。まずは`Option`です。

### `Option`列挙型とそのNull値に対する利点

前のセクションではIpAddr列挙型がRustの型システムを使ってプログラムのデータだけでなくより多くの情報をエンコードする方法を見てきました。このセクションでは、標準ライブラリによって定義された別の列挙型である`Option`のケーススタディについて説明します。 `Option`型は値が何かになる可能性がある、あるいは何もできないという非常に一般的なシナリオをエンコードするので、多くの場所で使用されます。この概念を型システムの観点から表現すると、コンパイラは処理する必要があるすべてのケースを処理したかどうかをチェックできます。この機能により、他のプログラミング言語で非常に一般的なバグを防ぐことができます。

プログラミング言語の設計はどの機能を含むかという点ではよく考えられますが除外する機能も重要です。Rustには他の多くの言語が持つNull機能はありません。*Null*はそこに値がないことを意味する値です。nullを持つ言語では、変数は常にnullまたはnot-nullの2つの状態のいずれかになります。

2009年のプレゼンテーション「Null References：The Billion Dollar Mistake」では、nullの発明者であるTony Hoareがこう言っています。

> それは10億ドルにも相当する私の誤りだ。null参照を発明したのは1965年のことだった。当時、私はオブジェクト指向言語 (ALGOL W) における参照のための包括的型システムを設計していた。目標は、コンパイラでの自動チェックで全ての参照が完全に安全であることを保証することだった。しかし、私は単にそれが容易だというだけで、無効な参照を含める誘惑に抵抗できなかった。これは、後に数え切れない過ち、脆弱性、システムクラッシュを引き起こし、過去40年間で10億ドル相当の苦痛と損害を引き起こしたとみられる。

ull値の問題は、null値をnull値として使用しようとすると、何らかのエラーが発生することです。このnullまたはnot-nullプロパティは広く使用されているため、この種のエラーは非常に簡単です。

しかし、nullが表現しようとしている概念はまだ有用なものです。nullは何らかの理由で現在無効であるか存在しない値です。

問題は実際にはコンセプトではなく、特定の実装である。 そのため、Rustにはnullはありませんが、値の存在または不在をエンコードできる列挙型があります。この列挙型は`Option <T>`であり、[標準ライブラリで定義されたもの][option]<!-- ignore -->では次のようになります。

[option]: ../../std/option/enum.Option.html

```rust
enum Option<T> {
    Some(T),
    None,
}
```

`Option<T>`列挙型は非常に有用であり、それを明示的にスコープに入れる必要はありません。加えてそのバリアントもそうです。`Some`と`None`を`Option::`プレフィックスなしで直接使うことができます。`Option <T>`列挙型はまだ普通の列挙型であり、`Some(T)`と`None`は`Option<T>`型のバリアントです。

`<T>`構文はまだ説明していないRustの特徴です。これはジェネリック型引数です。ジェネリックについては第10章で詳しく説明します。今のところ、`<T>`は`Option`列挙型の`Some`バリアントが一つのピースを保持できることを意味します。任意の型のデータの数値型と文字列型を保持するための`Option`値の使用例をいくつか示します。

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

`Some`ではなく`None`を使用する場合、`Some`バリアントが保持する型をコンパイラが推測することができないので、`Option<T>`の型をRustに伝える必要があります。

`Some`値があるとき値が存在しその値が`Some`内に保持されていることがわかります。ある意味で「None」という値があるときは、有効な値がないためnullと同じ意味です。では、なぜ`Option<T>`はnullよりも優れているのでしょうか？

`Option<T>`と`T`（ここで`T`は任意の型とすることができる）は異なる型であるため（`Option<i8>`に`i8`を追加しようとしている）、このコードはコンパイルされません。

```rust,ignore,does_not_compile
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

このコードを実行すると、次のようなエラーメッセージが表示されます。

```text
error[E0277]: the trait bound `i8: std::ops::Add<std::option::Option<i8>>` is
not satisfied
 -->
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + std::option::Option<i8>`
  |
```

このエラーメッセージは異なる型のため`i8`と`Option<i8> `の加算ができないことを意味します。Rustに`i8`のような型の値があるとき、コンパイラは常に正しい値を持つことを保証します。その値を使用する前に、nullをチェックする必要はありません。`Option<i8>`（あるいは扱っている値の種類）があるときだけ、値を持たない可能性について心配する必要があります。コンパイラは値を使う前にそのケースを処理するようにします。

つまり、`Option<T>`を`T`に変換してから`T`演算を実行する必要があります。一般に、これはnullに関する最も一般的な問題の1つをキャッチするのに役立ちます。実際には何かがnullではないと仮定します。

値がNULLではないと心配する必要はなく、コードに自信を持たせることができます。nullになる可能性のある値を持つためには、その値の型を`Option<T>`にすることによって明示的にオプトインする必要があります。次に、その値を使用するときは、値がnullの場合に明示的に処理する必要があります。値に`Option<T>`以外の型がある場合は、値がnullではないとみなすことができます。これは、Rustがnullの普及を制限し、Rustコードの安全性を高めるための意図的な設計決定でした。

そのため`Option<T>`型の値を持っているときに、`Some`バリアントから`T`値をどのように取得すれば、その値を使うことができるのでしょうか？`Option<T>`列挙型は、さまざまな状況で便利な多数のメソッドを持っています。[このドキュメント][docs]<!-- ignore -->でそれらをチェックすることができます。`Option<T>`のメソッドに慣れると、非常に役立ちます。

[docs]: ../../std/option/enum.Option.html

`Option<T>`の値を使うためには、それぞれのバリアントを扱うコードが必要です。`Some(T)`値を持っているときにのみ実行されるいくつかのコードを必要とし、このコードは`T`を使用することができます。`None`の値を持っていて、そのコードに`T`の値がない場合、他のコードを実行します。`match`式は、列挙型で使用されるときにこれを行うコントロールフロー構造です。列挙型のバリアントに応じて異なるコードを実行し、一致する値の中でデータを使用することができます。
