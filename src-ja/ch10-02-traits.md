## トレイト: 共通の振る舞いを定義する

*トレイト*は特定の型が持つ機能についてRustコンパイラに指示し、他の型と共有することができます。トレイトを使用して抽象的な方法で共有動作を定義することができます。トレイト境界を使用して、ジェネリックスが特定の動作を持つ任意のタイプであることを指定できます。

> 注：トレイトは、いくつかの違いがありますが他の言語では*interfaces*と呼ばれる機能に似ています。


### トレイを定義する

型の振る舞いは、その型に対して呼び出すことができるメソッドから構成されます。異なる型は、それらの型すべてに対して同じメソッドを呼び出せる場合、同じ動作を共有します。トレイトの定義は、メソッドシグニチャをグループ化して、目的を達成するために必要な一連の動作を定義する方法です。

たとえば、さまざまな種類と量のテキストを保持する複数の構造体があるとしましょう。特定の場所に保管されているニュース記事を保持する`NewsArticle`構造体と、新規ツイートか、リツイートか、はたまた他のツイートへのリプライなのかを示すメタデータを伴う最大で280文字までの`Tweet`構造体です。

`NewsArticle`や`Tweet`インスタンスに保存される可能性のあるデータの要約を表示できるメディア要約ライブラリを作成したいと考えています。これを行うには、それぞれの型から要約が必要です。インスタンスに対して`summarize`メソッドを呼び出すことによって要約を要求する必要があります。リスト10-12は、この振る舞いを表現する`Summary`トレイトの定義を示しています。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

<span class="caption">Listing 10-12: `summarize`メソッドで提供される振る舞いからなる`Summary`トレイト</span>

ここでは、`trait`キーワードを使用してトレイトを宣言し、次にこのトレイト名`Summary`を指定しています。中括弧の中で、このトレイトを実装する型の振る舞いを記述するメソッドシグニチャを宣言します。今回の場合は`fn summarize(&self) -> String`です。

メソッドシグニチャの後に、中括弧で実装する代わりに、セミコロンを使用します。このトレイトを実装する各タイプは、メソッドの本体に対して独自のカスタム動作を提供する必要があります。コンパイラは`Summary`トレイトを持つ型は、このシグニチャで定義された`summarize`メソッドを正確に定義するように強制します。

トレイトは、その本体に複数のメソッドを持つことができます。メソッドのシグニチャは1行に1つずつリストされ、各行はセミコロンで終わります。

### トレイトを型に実装する

`Summary`トレイトを使って望ましい振る舞いを定義しましたので、メディア要約の型に実装することができます。リスト10-13は見出し、著者、場所を使用して`summarize`の戻り値を生成する`NewsArticle`構造体の`Summary`トレイト実装を示しています。`Tweet`構造体の場合、ツイートの内容が既に280文字に制限されていると想定して、`summarize`をユーザ名にツイート全体のテキストが続く形で定義します。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# pub trait Summary {
#     fn summarize(&self) -> String;
# }
#
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

<span class="caption">リスト 10-13: `Summary`トレイトをN`ewsArticle`と`Tweet`型に実装する</span>

ある型の特性を実装することは、通常のメソッドを実装することと似ています。違いは、`impl`の後に実装したいトレイト名を入れてから、`for`キーワードを使い、そのトレイトを実装する型の名前を指定することです。`impl`ブロック内で、トレイト定義が定義したメソッドのシグニチャを記述します。各シグニチャの後にセミコロンを追加する代わりに、中括弧を使用して、メソッド本体に特定の型のトレイトのメソッドに欲しい特定の振る舞いを記述します。

このトレイトを実装した後、次のような通常のメソッドと同じ方法で、`NewsArticle`と`Tweet`のインスタンスのメソッドを呼び出すことができます。

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

このコードは`1 new tweet: horse_ebooks: of course, as you probably already know, people`と出力します。

リスト10-13の同じ*lib.rs*に`Summary`特性と`NewsArticle`と`Tweet`型を定義したので、それらはすべて同じスコープに入っています。この*lib.rs*を`aggregator`と呼ばれるクレート専用にして、誰か他の人がこのクレートの機能を活用して自分のライブラリのスコープに定義された構造体に`Summary`トレイトを実装したいとしましょう。まず、トレイトをスコープにインポートする必要があります。`use aggregator::Summary;`と指定してそれを行い、これにより自分の型に`Summary`を実装することが可能になります。`Summary`トレイトは、 他のクレートが実装するためには、公開トレイトである必要があり、ここではリスト10-12の`trait`の前に、`pub`キーワードを置いたのでそうなっています。

トレイトの実装で注意すべき制限の1つは、トレイトか対象の型が自分のクレートにローカルである時のみ、型に対してトレイトを実装できるということです。たとえば、`Tweet`は`aggregator`クレートのローカルなので、`Display`のような標準ライブラリのトレイトを`Tweet`のようなカスタムタイプに実装することができます。`Summary`は`aggregator`クレートのローカルなので、`aggregator`クレートに`Vec<T>`に`Summary`を実装することもできます。

しかし、外部の型に外部のトレイトを実装することはできません。例えば、`aggregator`クレート内で`Vec<T>`に対して`Display`トレイトを実装することはできません。`Display`と`Vec<T>`は標準ライブラリで定義され、`aggregator`クレートにローカルではないからです。この制限は、*コヒーレンス(coherence)*と呼ばれるプログラムのプロパティの一部であり、より具体的には、*オーファンルール(orphan rule)*であり、親の型が存在しないためそのように名前が付けられています。このルールは、他の人のコードが自分のコードを壊すことができないようにします。このルールがなければ、2つのクレートが同じ型の同じトレイトを実装でき、Rustはどちらの実装を使用すべきかがわからなくなります。

### デフォルト実装

場合によっては、全ての型の全メソッドに対して実装を必要とするのではなく、トレイトの全てあるいは一部のメソッドに対してデフォルトの振る舞いがあると便利です。そうすれば、特定の型のトレイトを実装する際に、各メソッドのデフォルト動作を保持またはオーバーライドできます。

リスト10-14は、リスト10-12のように、メソッドのシグニチャのみを定義するのではなく、`Summary`トレイトの`summarize`メソッドにデフォルトの文字列を指定する方法を示しています。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

<span class="caption">リスト 10-14: `summarize`メソッドのデフォルト実装がある`Summary`トレイトの定義</span>

デフォルト実装を使用して独自の実装を定義するのではなく、`NewsArticle`のインスタンスをまとめるには、`impl Summary for NewsArticle {}`と空の`impl`ブロックを指定します。

`NewsArticle`に`summarize`メソッドを直接定義しなくても、デフォルトの実装を提供し、`NewsArticle`が`Summary`トレイトを実装するように指定しました。その結果、`NewsArticle`のインスタンスに対して`summarize`メソッドを呼び出すことができます。

```rust,ignore
let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from("The Pittsburgh Penguins once again are the best
    hockey team in the NHL."),
};

println!("New article available! {}", article.summarize());
```

このコードは`New article available! (Read more...)`と出力します。

`summarize`のデフォルト実装を作成しても、リスト10-13の`Tweet`の`Summary`の実装について何も変更する必要はありません。その理由は、既定の実装をオーバーライドする構文は、既定の実装を持たないトレイトのメソッドを実装するための構文と同じだからです。

デフォルト実装は、他のデフォルト実装がないメソッドでも呼び出すことができます。このように、トレイトは多くの有用な機能を提供することができ、実装者が一部しか指定しなくてよくなります。例えば、`summary`トレイトを実装する必要がある`summarize_author`メソッドを定義し、`summarize_author`メソッドを呼び出すデフォルト実装を持つ`summarize`メソッドを定義することができます。

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

このバージョンの`Summary`を使うためには、型にトレイトを実装するとき`summarize_author`のみを定義する必要があります。

```rust,ignore
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

`summarize_author`を定義した後、`Tweet`構造体のインスタンスに対して`summarize`を呼び出すことができます。`summarize`のデフォルト実装は`summarize_author`の定義を呼び出します。`summarize_author`を実装しているので、`Summary`トレイとは、コードを書く必要なしに`summarize`メソッドの動作を与えました。

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

このコードは`1 new tweet: (Read more from @horse_ebooks...)`と出力します。

同じメソッドのオーバーライド実装からデフォルト実装を呼び出すことはできません。

### 引数としてのトレイト

トレイトを定義し、それらを型に実装する方法を知っているので、さまざまな型の引数を受け入れるためにトレイトを使用する方法を探ることができます。

たとえば、リスト10-13では`NewsArticle`と`Tweet`型の`Summary`トレイトを実装しました。引数に`summary`トレイトを実装した型の`item`を受け取り、`item`の`summarize`メソッドを呼び出す`notify`関数を定義することができます。これを行うには、次のように `impl Trait`構文を使用できます。

```rust,ignore
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

`notify`の本体では`summar`のような`summary`トレイトから来た`item`のメソッドを呼び出すことができます。

#### トレイト境界

`impl Trait`構文は短い例では便利ですが、長い形式ではシンタックスシュガーです。これは*トレイト境界*と呼ばれ、以下のようになります。

```rust,ignore
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

これは上記の例と同じですが、少し冗長です。コロンと内側の山括弧の後に、ジェネリック型引数の宣言でトレイト境界を配置します。`T`に束縛されたトレイトのため、`NewsArticle`や`Tweet`のインスタンスを引数に渡して`notify`を呼ぶことができます。`String`や`i32`のような他の型の関数を呼び出すコードは、`Summary`を実装していないのでコンパイルされません。

`Imp Trait`ではなく、いつこの構文を使うべきでしょうか？`impl Trait`は短い例題にはうってつけですが、trait境界はもっと複雑なものに便利です。たとえば、`Summary`を実装する2つの引数を受け取りたいとします。

```rust,ignore
pub fn notify(item1: impl Summary, item2: impl Summary) {
```

これは`item1`と`item2`が異なる型を持つことが許されていれば（両方とも`Summary`を実装している限り）うまくいくでしょう。しかし、両方を同じタイプにすることを強制したいのであればどうしますか？それは、トレイト境界を使用する場合にのみ可能です。

```rust,ignore
pub fn notify<T: Summary>(item1: T, item2: T) {
```

#### `+`で複数のトレイトを指定する

`item`に書式を表示するために`notify`メソッドが必要で、その中で`summarize`メソッドを使うのであれば、`item`は`Display`と`Summary`の二つの異なるトレイトを同時に実装する必要があります。これは`+`構文を使って行うことができます。

```rust,ignore
pub fn notify(item: impl Summary + Display) {
```

この構文は、ジェネリック型のトレイト境界でも有効です。

```rust,ignore
pub fn notify<T: Summary + Display>(item: T) {
```

#### より明確なコードのための `where`句

しかし、あまりにも多くのトレイト境界を使用することには欠点があります。各ジェネリックスには独自のトレイト境界があるため、複数のジェネリック型引数を持つ関数は、関数名とその引数リストの間に多くのトレイト境界情報を持ち、関数シグニチャを読みにくくします。この理由から、Rustは関数シグニチャの後に`where`節の中でトレイト境界を指定するための代替構文を持っています。次のように書くのではなく、

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

次のように`where`節を使うことができます。

```rust,ignore
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

関数名、引数リスト、および戻り値の型が密接して定義されているので、この関数のシグニチャはすっきりします。

### トレイトを返す

戻り値の位置で`impl Trait`構文を使用して、トレイトを実装したものを返すこともできます。

```rust,ignore
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

このシグニチャは、「私は`Summary`トレイトを実装するものを返すつもりですが、正確な型を伝えるつもりはありません」と述べています。この場合、`Tweet`を返していますが、呼び出した人はそれを知りません。

これはどのように役に立つのでしょうか？13章では、トレイトに大きく依存する2つの機能、クロージャー、イテレーターについて学びます。これらの機能は、コンパイラだけが知っている型、または非常に長い型を作成します。`impl Trait`は長い型を書く必要なく、「これは`Iterator`を返します」と簡単に宣言することができます。

しかし、これは戻ってくる単一の型を持っている場合にのみ機能します。たとえば、これはうまく*いかない*でしょう。

```rust,ignore,does_not_compile
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from("Penguins win the Stanley Cup Championship!"),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from("The Pittsburgh Penguins once again are the best
            hockey team in the NHL."),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from("of course, as you probably already know, people"),
            reply: false,
            retweet: false,
        }
    }
}
```

ここでは、`NewsArticle`または`Tweet`のいずれかを返します。これは、`impl Trait`がどのように機能するかという制限があるため、うまくいきません。このコードを書くには、第17章の「さまざまな型の値を許容する特性オブジェクトの使用」の項まで待つ必要があります。

### トレイト境界でlargest関数を修正する

ジェネリック型引数の境界を使用して振る舞いを指定する方法を知ったので、リスト10-5に戻ってジェネリック型引数を使用する`largest`関数の定義を修正しましょう。前回このコードを実行しようとすると、以下のようなエラーが発生しました。

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:12
  |
5 |         if item > largest {
  |            ^^^^^^^^^^^^^^
  |
  = note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

`largest`の本体では、大なり(`>`)演算子を使って`T`型の2つの値を比較したいと考えました。この演算子は標準ライブラリトレイト`std:: cmp::PartialOrd`のデフォルトメソッドとして定義されているため、比較できるあらゆる型のスライスに対して動くようにTのトレイト境界に`PartialOrd`を指定する必要があります。`PartialOrd`は、初期化処理に入っているので、スコープに入れる必要はありません。`largest`のシグニチャを次のように変更します。

```rust,ignore
fn largest<T: PartialOrd>(list: &[T]) -> T {
```

コードをコンパイルすると、異なる一連のエラーが発生します。

```text
error[E0508]: cannot move out of type `[T]`, a non-copy slice
 --> src/main.rs:2:23
  |
2 |     let mut largest = list[0];
  |                       ^^^^^^^
  |                       |
  |                       cannot move out of here
  |                       help: consider using a reference instead: `&list[0]`

error[E0507]: cannot move out of borrowed content
 --> src/main.rs:4:9
  |
4 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
```

このエラーのキーとなる行は`cannot move out of type [T], a non-copy slice`です。ジェネリックでないバージョンのlargest関数では、最大のi32かcharを探そうとするだけでした。第4章の「スタックオンリーデータ：コピー」で説明したように、既知のサイズを持つ`i32`や`char`のような型はスタックに格納できるので、`Copy`トレイトを実装します。しかし、`largest`関数をジェネリックスにすることで、`list`引数は`Copy`トレイトを実装しない型を持つことが可能になりました。その結果、`list[0]`から`maximum`変数に値を移動することができなくなり、このエラーが発生します。

`Copy`トレイトを実装する型だけでこのコードを呼び出すには、`T`のトレイト境界に`Copy`を追加することで実現します。リスト10-15は、関数に渡したスライスの値の型が`i32`や`char`などのように、`PartialOrd`と`Copy`を実装する限り、コンパイルできるジェネリックな`largest`関数の完全なコードを示しています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
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

<span class="caption">リスト 10-15: `PartialOrd`と`Copy`トレイトを実装するあらゆるジェネリックな型に対して動く、`largest`関数の定義</span>

もしlargest関数をCopyを実装する型だけに制限したくなかったら、Copyではなく、 TがCloneというトレイト境界を含むと指定することもできます。そうすると、`largest`関数に所有権を持たせたいときに、スライス内の各値を複製することができます。`clone`関数を使うことは、`String`のようなヒープデータを所有する型の場合にヒープ割り当てを増やすことを意味し、大量のデータを扱う場合はヒープ割り当てが遅くなる可能性があります。

`largest`を実装する別の方法は、スライス内の`T`値への参照を返す関数です。戻り値の型を`T`ではなく`&T`に変更して、関数の本体を変更して参照を返すと、`Clone`や`Copy`トレイト境界は必要なくなり、ヒープ割り当てを避けることができます。これらの他の解決策を自分で実装してみてください！

### トレイト境界を使用して、メソッド実装を条件分けする

`impl`ブロックでジェネリックな型引数を使用するトレイト境界を活用することで、特定のトレイトを実装する型に対するメソッド実装を
条件分岐できます。たとえば、リスト10-16の`Pair<T>`型は常に`new`関数を実装しています。しかし、`Pair<T>`は、内部の型`T`が比較を可能にする`PartialOrd`トレイトと出力を可能にする`Display`トレイトを実装している時のみ、`cmp_display`メソッドを実装します。

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

<span class="caption">リスト 10-16: トレイト境界によってジェネリックな型に対するメソッド実装を条件分けする</span>

また、別のトレイトを実装する任意の型のための条件を条件付きで実装することもできます。トレイト境界を満たす任意のタイプの特性の実装は、*ブランケット実装(blanket implementation)*と呼ばれ、Rust標準ライブラリで広く使用されています。例えば、標準ライブラリは、`Display`トレイトを実装する任意の型の`ToString`トレイトを実装します。標準ライブラリの`impl`ブロックはこのコードに似ています。

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

標準ライブラリはこのブランケットの実装を持っているので、`Display`特性を実装するすべての型の`ToString`トレイトによって定義された`to_string`メソッドを呼び出すことができます。例えば、整数は`Display`を実装しているので、整数を対応する`String`値に変換することができます。

```rust
let s = 3.to_string();
```

ブランケット実装は、「実装したもの」セクションのトレイトに関するドキュメントに記載されています。

トレイトとトレイト境界は、複製を減らすためにジェネリック型引数を使用するコードを書くだけでなく、ジェネリック型に特定の動作を持たせたいというコンパイラを指定します。コンパイラは、トレイト境界の情報を使用して、コードで使用されているすべての具体的な型が正しい動作を提供しているかどうかをチェックできます。動的に型付けされた言語では、型が実装しなかった型に対してメソッドを呼び出すと、実行時にエラーが発生します。しかし、Rustはこれらのエラーをコンパイルするためにコンパイルするので、コードを実行する前に問題を修正する必要があります。さらに、コンパイル時にすでにチェックしているため、実行時に動作をチェックするコードを記述する必要はありません。そうすることで、ジェネリックの柔軟性を失うことなく、パフォーマンスが向上します。

もう使用したことのある別の種類のジェネリクスは、*ライフタイム*と呼ばれます。型が欲しい振る舞いを保持していることを保証するのではなく、必要な間だけ参照が有効であることをライフタイムは保証します。ライフタイムがどうやってそれを行うかを見ましょう。