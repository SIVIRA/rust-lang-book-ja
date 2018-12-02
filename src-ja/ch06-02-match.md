## `match`制御フロー演算子

Rustには非常に強力な制御フロー演算子`match`があり、一連のパターンと値を比較しどのパターンが一致するかに基づいてコードを実行することができます。パターンはリテラル値、変数名、ワイルドカード、その他多くのもので構成できます。第18章では、さまざまな種類のパターンとそのパターンについて説明します。`match`の威力は、パターンの表現力とコンパイラがすべての可能なケースが処理されることを確認するという事実から来ます。

`match`はコイン選別機のように考えることができます。コインはさまざまなサイズの穴があるトラックを滑り落ち、各コインは遭遇する最初の穴から落ちます。同様に値は`match`の各パターンを通り、最初のパターンでは「適合」の値は実行中に使用される関連コードブロックに入ります。

先ほどのコインについて言及したので、`match`を使ってコインを例にします。リスト6-3に示すように、未知の米国のコインを受け取ることができる関数を書くことができます。これは計数機と同様にコインがどれであるかを判定しその値をセントで返します。

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

<span class="caption">リスト 6-3: enumとそのパターンとしてenumのバリアントを持つ`match`式</span>

`value_in_cents`関数の`match`を分解してみましょう。まず、`match`キーワードとそれに続く式を列挙しますこの場合、値は`coin`です。これは`if`で使われる式と非常によく似ていますが、`if`では式がブール値を返す必要がありますが、ここでは任意の型にすることができます。この例の`coin`の型は、第1行で定義した`Coin`列挙型です。

次は `match`armです。armにはパターンとコードの2つの部分があります。最初のarmは`Coin::Penny`の値を持つパターンとパターンと実行するコードを分離する`=>`演算子です。この場合のコードはちょうど値`1`です。各armは次の腕とカンマで区切られています。

`match`式が実行されると、結果の値と各armのパターンが順番に比較されます。パターンが値と一致する場合、そのパターンに関連付けられたコードが実行されます。そのパターンが値と一致しない場合、コイン選別機と同様に次のarmに実行が継続されます。リスト6-3では、`match`には4つのarmがあります。

各armに関連付けられたコードは式であり、一致するarmの式の結果の値は`match`式全体に対して返される値です。

中括弧はmatch armコードが短い場合は通常は使用されません。スト6-3のように、各armが値を返すだけです。match armで複数行のコードを実行する場合は、中括弧を使用できます。たとえば、次のコードはメソッドが `Coin::Penny`で呼び出されるたびに`Lucky penny!`を出力しますが、ブロックの最後の値'1'を返します。

```rust
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter,
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### 値にバインドするパターン

match armのもう1つの便利な機能は、パターンにマッチする値の部分にバインドできることです。これは列挙型から値を抽出する方法です。

例として、enumバリアントの1つを内部に保持するように変更しましょう。1999年から2008年にかけて、米国は50の州ごとにデザインの異なる25セント硬貨を作りました。他の硬貨は国家がデザインをしていないため、25セント硬貨には特別な値があります。この情報を`enum`に追加するには、`Quarter`バリアントを変更して内部に格納された `UsState`値をインクルードするようにします。これはコードリスト6-4で行っています。

```rust
#[derive(Debug)] // So we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

<span class="caption">リスト 6-4: Quarterバリアントも`UsState`値を保持する`Coin`列挙型</span>

友人が全50州の25セント硬貨を収集しようとしているとしましょう。コインタイプで緩やかな変更を並べ替える間に、各四半期に関連する州の名前も呼び出すので、友達が持っていない場合はコレクションに追加することができます。

このコードのmatch式では、`Coin::Quarter`バリアントの値と一致するパターンに`state`という変数を追加します。`Coin::Quarter`が一致すると、`state`変数はその25セント硬貨の状態の値にバインドされます。次にそのようなコードのために`state`を使用することができます。

```rust
# #[derive(Debug)]
# enum UsState {
#    Alabama,
#    Alaska,
# }
#
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

`value_in_cents(Coin::Quarter(UsState::Alaska))`を呼び出す場合、`coin`は`Coin::Quarter(UsState::Alaska)`になります。その値を各match armと比較すると、`Coin::Quarter(state)`に達するまで一致するものはありません。その時点で、`state`のバインディングは`UsState::Alaska`の値になります。`println!`式でそのバインディングを使うことができるので、`Quarter`の`Coin`列挙型から内部状態値を取得できます。

### `Option<T>`とのマッチング

前のセクションでは、`Option<T>`を使うとき、`Some`の場合の内部の`T`値を得たいと思っていました。`Coin`列挙型で行ったように`match`を使って`Option<T>`を扱うこともできます。コインを比較するのではなく、`Option<T>`のバリエーションを比較しますが、 `match`式の働きは変わりません。

`Option<i32>`を引数に受けとる関数を書いて、内部に値があればその値に1を加えたいとしましょう。内部に値がない場合、関数は `None`値を返し、何も操作を行わないようにしなければなりません。

この関数は`match`のおかげで非常に書きやすく、リスト6-5のようになります。

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

<span class="caption">リスト 6-5: `Option<i32>`で`match`式を使う関数</span>

`plus_one`の最初の実行をもっと詳しく調べてみましょう。`plus_one(five)`を呼び出すと、`plus_one`の本体の変数`x`は `Some(5)`という値になります。次に、それを各match armと比較します。

```rust,ignore
None => None,
```

`Some(5)`の値はパターン`None`と一致しませんので次のarmに進みます。

```rust,ignore
Some(i) => Some(i + 1),
```

`Some(5)`は`Some(i)`にマッチします。私たちは同じバリアントを持っています。`i`は`Some`に含まれる値に束縛されるので、`i`は値 `5`をとります。マッチアーム内のコードが実行されるので、`i`の値に1を加えて内部に`6`という値を新しい`Some`値を作ります。

リスト6-5の`plus_one`の2番目の呼び出しを考えてみましょう。ここで`x`は`None`です。私たちは`match`に入り、最初のarmと比較します。

```rust,ignore
None => None,
```

これは一致します。追加する値はないのでプログラムは停止し、`=>`の右側に`None`値を返します。最初のarmが一致したので、他のarmは比較されません。

`match`とenumを組み合わせることは多くの状況で役に立ちます。このパターンはenumに対する`match`、変数を内部のデータにバインドし、それに基づいてコードを実行するなどRustコードに多く見られます。最初はややこしいですが、慣れてしまえばすべての言語でそれを使いたいと思うでしょう。

### matchの網羅性

私たちが議論する必要のある`match`のもう一つの側面があります。バグがありコンパイルされない`plus_one`関数のバージョンを考えてみましょう。

```rust,ignore,does_not_compile
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

`None`の場合の扱いがないのでこのコードはバグを引き起こします。このコードをコンパイルしようとすると、このエラーが発生します。

```text
error[E0004]: non-exhaustive patterns: `None` not covered
 -->
  |
6 |         match x {
  |               ^ pattern `None` not covered
```

Rustはあらゆる可能なケースをカバーしていないことを知っています。Rustのマッチは*網羅的*です。コードが有効であるためには、すべての可能性を排除しなければなりません。特に`Option<T>`の場合、Rustが明示的に`None`を扱うことを忘れないようにすると、nullを持っていると仮定することから守ります。

### `_`プレースホルダ

Rustにはすべての可能な値をリストしたくないときに使用できるパターンもあります。例えば、`u8`は0から255までの有効な値を持つことができます.1,3,5,7の値だけを扱う場合、0,2,4,6,7,8のいずれかをリストアップする必要はありません。その場合、特別なパターン`_`を代わりに使うことができます。

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

`_`パターンは任意の値と一致します。他の武器の後ろに置くことによって、`_`はその前に指定されていない可能性のあるすべてのケースに一致します。`()`は単なる単価であるので、`_`の場合は何も起こりません。結果として、`_`プレースホルダの前にリストされていないすべての可能な値に対して何もしたくないと言うことができます。

しかし、`match`式はケースの*one*だけを気にしている状況では違う記述をします。その場合、Rustは`if let`を提供します。
