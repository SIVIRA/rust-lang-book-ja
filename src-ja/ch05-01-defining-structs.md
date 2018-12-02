## 構造体を定義し、インスタンス化する

構造体は第3章で説明したタプルに似ています。タプルのように構造体の断片は異なる型にすることができます。タプルとは異なり、各データに名前を付けて、値の意味を明確にします。これらの名前の結果、構造体はタプルよりも柔軟性があります。つまり、インスタンスの値を指定またはアクセスするためにデータの順序に頼る必要はありません。

構造体を定義するためにキーワード`struct`を入力し、構造体全体の名前を付けます。構造体の名前は一緒にグループ化されるデータの重要性を記述する必要があります。次に中括弧の中でデータの名前と型を定義します。これは*fields*と呼ばれます。たとえば、リスト5-1はユーザーアカウントに関する情報を格納する構造体を示しています。

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

<span class="caption">リスト 5-1: `User`構造体の定義</span>

構造体を定義した後でそれを使用するには、各構造体の具体的な値を指定して構造体の*インスタンス*を作成します。構造体の名前を記述してインスタンスを作成し`key：value`ペアを含む中括弧を追加します。ここでキーはフィールドの名前であり、値はこれらのフィールドに格納するデータです。構造体で宣言したのと同じ順序でフィールドを指定する必要はありません。言い換えれば、構造体の定義は型の一般的なテンプレートのようであり、インスタンスは型の値を作成するための特定のデータでそのテンプレートを埋めます。たとえば、リスト5-2に示すように特定のユーザーを宣言することができます。

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

<span class="caption">リスト 5-2: `User`構造体のインスタンスを作成する</span>

構造体から特定の値を取得するには、ドット表記法を使用できます。このユーザの電子メールアドレスだけが必要な場合は、この値を使用したい場合はいつでも`user1.email`を使用できます。インスタンスが変更可能な場合は、ドット表記を使用して特定のフィールドに値を割り当てることで値を変更できます。リスト5-3は変更可能な`User`インスタンスの`email`フィールドの値を変更する方法を示しています。

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```

<span class="caption">リスト 5-3: `User`インスタンスの`email`フィールドの値を変更する</span>

インスタンス全体は変更可能でなければならないことに注意してください。Rustは特定のフィールドだけを変更可能としてマークすることはできません。

任意の式と同様に関数本体の最後の式として構造体の新しいインスタンスを構築して、その新しいインスタンスを暗黙的に返すことができます。コードリスト5-4は与えられた電子メールとユーザ名を持つ`User`インスタンスを返す`build_user`関数を示しています。`active`フィールドは`true`の値をとり、`sign_in_count`は`1`の値を得ます。

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">リスト 5-4: emailとusernameをとり、`User`インスタンスを返す`build_user`関数</span>

構造体フィールドと同じ名前の関数パラメータに名前を付けるのは理にかなっていますが、`email`と`username`フィールド名と変数を繰り返さなければならないのはちょっと面倒です。構造体にさらにフィールドがある場合は、それぞれの名前を繰り返すとさらに複雑になります。そのため便利な略記があります。

### フィールド初期化の使用変数とフィールドの名前が同じ場合の省略形

パラメータ名と構造体フィールド名はリスト5-4と全く同じですので、*field init shorthand*構文を使用して`build_user`を書き直してまったく同じように動作しますが、リスト5-5に示すように`email`と`username`を呼び出します。


```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">リスト 5-5: `email`と`username`パラメータがstructフィールドと同じ名前を持つので、フィールドinitを使う`build_user`関数</span>

ここでは、`email`という名前のフィールドを持つ`User`構造体の新しいインスタンスを作成しています。`email`フィールドの値を`build_user`関数の`email`パラメータの値に設定します。`email`フィールドと`email`パラメータは同じ名前なので、 `email：email`ではなく`email`を書くだけです。

### 構造体のアップデート構文を使用して他のインスタンスからインスタンスを作成する

古いインスタンスの値の大部分を使用するが、いくつかを変更する構造体の新しいインスタンスを作成すると便利なことがよくあります。*struct update syntax*を使用してこれを行います。

まず、リスト5-6はアップデート構文なしで`user2`に新しい`User`インスタンスを作成する方法を示しています。`email`と`username`に新しい値を設定しますが、リスト5-2で作成した`user1`と同じ値を使用します。

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
```

<span class="caption">リスト 5-6: `user1`からのいくつかの値を使って新しい` User`インスタンスを作成する</span>

構造体のアップデート構文を使用すると、リスト5-7に示すようにコードを減らして同じ効果を得ることができます。`..`構文は、明示的に設定されていない残りのフィールドは、指定されたインスタンスのフィールドと同じ値を持つ必要があることを指定します。

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

<span class="caption">リスト 5-7: struct update構文を使って`User`インスタンスの新しい`email`と`username`値を設定しますが、`user1`変数のインスタンスのフィールドから残りの値を使います</span>

コードリスト5-7のコードでは、`email2`に`email`と`username`とは異なる値を持つインスタンスを作成しますが、`user1`の `active`と`sign_in_count`フィールドの値は同じです。

### 異なる型を作成するための名前付きフィールドのないタプル構造体

*tuple structs*と呼ばれるタプルに似た構造体を定義することもできます。タプル構造体には、構造体名が提供する追加の意味がありますが、フィールドに関連付けられた名前はありません。むしろ、彼らはフィールドのタイプを持っています。タプル構造体は、タプル全体に名前を付け、タプルを他のタプルとは異なる型にする場合や、通常の構造体のように各フィールドの名前を冗長または冗長にする場合に便利です。

タプル構造体を定義するには`struct`キーワードと構造体名の後にタプルの型を続けます。例えば`Color`と`Point`という2つのタプル構造体の定義と使用法を次に示します。

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

`black`と`origin`値は異なるタプル構造体のインスタンスであるため、異なる型であることに注意してください。構造体内のフィールドに同じ型があっても、定義する各構造体は独自の型です。たとえば、`Color`型のパラメータをとる関数は、両方の型が3つの` i32`値で構成されていても、引数として`Point`を取ることはできません。さもなければ、タプル構造体インスタンスはタプルのように振る舞います。それらを個々の部分に分解することができます。個々の値にアクセスするには、インデックスのあとに `.`を使用できます。

### 任意のフィールドを持たないUnitのような構造体

また、フィールドを持たない構造体を定義することもできます。これらは、ユニット型の`()`と同じように動作するため、*unit-like structs*と呼ばれます。Unitのような構造体はある型の型を実装する必要がありますが、型自体に格納するデータはありません。第10章でトレイトについて議論します。

> ### 構造データの所有権
>
> リスト5-1の`User`構造体定義では、`&str`文字列スライス型ではなく所有された`String`型を使用しました。この構造体のインスタンスがすべてのデータを所有し構造体全体が有効である限り、そのデータが有効であることが望ましいため、これは意図的な選択です。
>
> 構造体が他のものが所有するデータへの参照を格納することは可能ですが、構造体がある限り第10章で説明するRust機能である*lifetimes*を使用する必要があります。lifetimeを指定せずに構造体の参照を格納すると動作しません。
>
> <span class="filename">ファイル名: src/main.rs</span>
>
> ```rust,ignore,does_not_compile
> struct User {
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
>     active: bool,
> }
>
> fn main() {
>     let user1 = User {
>         email: "someone@example.com",
>         username: "someusername123",
>         active: true,
>         sign_in_count: 1,
>     };
> }
> ```
>
> コンパイラはlifetime指定子が必要であるとエラーを出します。
>
> ```text
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 2 |     username: &str,
>   |               ^ expected lifetime parameter
>
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 3 |     email: &str,
>   |            ^ expected lifetime parameter
> ```
>
> 第10章ではこれらのエラーを修正して参照を構造体に格納する方法について説明しますが、現時点では`&str`のような参照の代わりに`String`のような所有型を使ってエラーを修正します。
