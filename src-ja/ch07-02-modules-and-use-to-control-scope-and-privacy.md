## スコープとプライバシーを制御するモジュールシステム

Rustには「モジュールシステム」と呼ばれる機能がありますが、モジュールにもいくつかの機能があります

* モジュール、コードを整理し、パスのプライバシーを制御する方法
* パス、アイテムに名前を付ける方法
* パスをスコープに入れるための`use`キーワード
* `pub`はアイテムを公開するためのキーワードです
* アイテムを`as`キーワードでスコープに入れるときにアイテムの名前を変更する
* 外部パッケージを使用する
* 大きな`use`リストをクリーンアップするネストされたパス
* glob演算子を使用して、モジュール内のすべてをスコープに入れる
* モジュールを個々のファイルに分割する方法

モジュールはコードをグループにまとめることができます。コードリスト7-1は`guitar`という名前の関数を含む`sound`モジュールを定義するコードの例です。

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod sound {
    fn guitar() {
        // Function body code goes here
    }
}

fn main() {

}
```

<span class="caption">リスト 7-1: `sound`モジュールは`guitar`関数と`main`関数を含んでいます</span>

2つの関数`guitar`と`main`を定義しました。`mod`ブロック内で`guitar`関数を定義しました。このブロックは `sound`という名前のモジュールを定義します。

コードをモジュールの階層に編成するにはリスト7-2に示すように他のモジュールの内部にモジュールをネストできます。

<span class="filename"> ファイル名: src/main.rs</span>

```rust
mod sound {
    mod instrument {
        mod woodwind {
            fn clarinet() {
                // Function body code goes here
            }
        }
    }

    mod voice {

    }
}

fn main() {

}
```

<span class="caption">リスト 7-2: モジュール内のモジュール</span>

この例では、リスト7-1と同じ方法で`sound`モジュールを定義しました。次に、`sound`モジュール内に`instrument`と`voice`という2つのモジュールを定義しました。`instrument`モジュールには別のモジュール`woodwind`が定義されており、そのモジュールには`clarinet`という名前の関数が含まれています。

*src/main.rs*と*src/lib.rs*は*クレートルート*と呼ばれている「ライブラリと実行可能ファイルを作るためのパッケージとクレート」セクションで述べました。これらの2つのファイルのいずれかの内容は、クレートのモジュールツリーのルートに`crate`という名前のモジュールを形成するので、クレートルートと呼ばれます。リスト7-2には、リスト7-3のようなモジュールツリーがあります。

```text
crate
 └── sound
     └── instrument
        └── woodwind
     └── voice
```

<span class="caption">リスト 7-3: リスト7-2のコードのモジュールツリー</span>

このツリーは、モジュールのいくつかが互いに内部でどのように入れ子になっているかを示しています（`instrument`の中に`woodwind`ネストなど）、そしていくつかのモジュールがどのように兄弟同士であるか（`instrument`と`voice`は`sound`で定義されています。モジュールツリー全体は`crate`という暗黙のモジュールの下にあります。

このツリーはコンピュータ上にあるファイルシステムのディレクトリツリーを思い出させるかもしれません。ファイルシステムのディレクトリと同様に、望む組織を作成するモジュールの中にコードを配置します。もう1つの類似点は、ファイルシステムまたはモジュールツリー内の項目を参照するには、*path*を使用することです。

### モジュールツリー内のアイテムを参照するためのパス

関数を呼び出す場合は、その*path*を知る必要があります。「パス」は「名前」の同義語ですが、ファイルシステムのメタファを呼び起こします。さらに、関数、構造体、およびその他の項目は、同じ項目を参照する複数のパスを持つことがあるため、「名前」はあまり適切な概念ではありません。

*パス*には次の2つの形式があります。

* *絶対パス*はクレート名かリテラル`crate`を使ってクレートルートから始まります。
* *相対パス*は、現在のモジュールから始まり、`self`、`super`、または現在のモジュールの識別子を使用します。

絶対パスと相対パスの後には、二重コロン（`::`）で区切られた1つ以上の識別子が続きます。

コードリスト7-2の`main`関数で`clarinet`関数をどのように呼び出すのですか？つまり、`clarinet`関数のパスは何ですか？リスト7-4では、いくつかのモジュールを削除してコードを少し簡略化し、`main`から`clarinet`関数を呼び出す2つの方法を示します。この例はまだコンパイルされませんが、少し説明します。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
mod sound {
    mod instrument {
        fn clarinet() {
            // Function body code goes here
        }
    }
}

fn main() {
    // Absolute path
    crate::sound::instrument::clarinet();

    // Relative path
    sound::instrument::clarinet();
}
```

<span class="caption">リスト 7-4: 絶対パスと相対パスを使用して`main`関数から単純化されたモジュールツリーで`clarinet`関数を呼び出す</span>

`main`関数からクラリネット関数を呼び出す最初の方法は、絶対パスを使います。`clarinet`は`main`と同じクレート内で定義されているので、`crate`キーワードを使用して絶対パスを開始します。 次に`clarinet`関数に進むまで、それぞれのモジュールをインクルードします。  これは`/sound/instrument/clarinet`というパスを指定してコンピュータ上のその場所でプログラムを実行するのと同様です。クレートルートから始めるために`crate`という名前を使うのは、`/`を使ってシェルのファイルシステムルートから始めるのと同じです。

`main`関数からクラリネット関数を呼び出す2番目の方法は相対パスを使います。パスは`sound`という名前で始まります。モジュールは`main`関数と同じレベルのモジュールツリーで定義されています。これは、コンピュータのその場所でプログラムを実行するためのパス `sound/instrument/clarinet`を指定するのと同様です。名前で始まる場合は、パスが相対パスであることを意味します。

リスト7-4はまだコンパイルされませんが、コンパイルしてみましょう。リスト7-5にエラーを示します。

```text
$ cargo build
   Compiling sampleproject v0.1.0 (file:///projects/sampleproject)
error[E0603]: module `instrument` is private
  --> src/main.rs:11:19
   |
11 |     crate::sound::instrument::clarinet();
   |                   ^^^^^^^^^^

error[E0603]: module `instrument` is private
  --> src/main.rs:14:12
   |
14 |     sound::instrument::clarinet();
   |            ^^^^^^^^^^
```

<span class="caption">リスト 7-5: リスト7-4のコード作成によるコンパイラエラー</span>

エラーメッセージではモジュール`instrument`はプライベートであると言いっています。`instrument`モジュールと`clarinet`関数の正しいパスがあることがわかりますが、プライベートなので使用できません。

### プライバシー境界としてのモジュール

ここまででモジュールの構文について説明しそれらを体系的に使用することができました。Rustにはモジュールがあります。モジュールはRustの*プライバシー境界*です。関数や構造体のような項目をプライベートにしたい場合はそれをモジュールに入れます。プライバシールールは次のとおりです。

* すべての項目（関数、メソッド、構造体、列挙型、モジュール、annd定数）はデフォルトではプライベートです
* `pub`キーワードを使って項目を公開することができます
* 現在のモジュールの子であるモジュールで定義されたプライベートコードを使用することはできません
* 祖先モジュールまたは現在のモジュールで定義されたコードを使用することができます

言い換えれば、`pub`キーワードを持たない項目は、現在のモジュールからモジュールツリーを見る際にプライベートですが、`pub`キーワードのない項目は、現在のモジュールからツリーを`見上げる`ときにpublicです。また、ファイルシステムを考えてみましょう。ディレクトリへのアクセス権がない場合は、親ディレクトリからそのディレクトリを調べることはできません。ディレクトリへのアクセス権を持っている場合、そのディレクトリとその祖先ディレクトリを見ることができます。

###`pub`キーワードを使ってアイテムを公開する

リスト7-5のエラーは、`instrument`モジュールはプライベートであると言いました。`instrument`モジュールに`pub`キーワードを付けて、 `main`関数から使用できるようにしましょう。この変更はリスト7-6に示されていますが、まだコンパイルされませんが、別のエラーが発生します：

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore,does_not_compile
mod sound {
    pub mod instrument {
        fn clarinet() {
            // Function body code goes here
        }
    }
}

fn main() {
    // Absolute path
    crate::sound::instrument::clarinet();

    // Relative path
    sound::instrument::clarinet();
}
```

<span class="caption">リスト 7-6: `instrument`モジュールを`pub`と宣言して`main`から使用できるようにしました</span>

`mod instrument`の前に`pub`キーワードを追加すると、モジュールが公開されます。この変更により、`sound`にアクセスすることができれば、`instrument`にアクセスすることができます。`instrument`の内容はまだプライベートです。モジュールをパブリックにしても内容は公開されません。モジュールの`pub`キーワードは親モジュールのコードを参照します。

ただし、リスト7-6のコードでは、コードリスト7-7に示すようにエラーが発生します。

```text
$ cargo build
   Compiling sampleproject v0.1.0 (file:///projects/sampleproject)
error[E0603]: function `clarinet` is private
  --> src/main.rs:11:31
   |
11 |     crate::sound::instrument::clarinet();
   |                               ^^^^^^^^

error[E0603]: function `clarinet` is private
  --> src/main.rs:14:24
   |
14 |     sound::instrument::clarinet();
   |                        ^^^^^^^^
```

<span class="caption">リスト 7-7: コードリスト7-6のココンパイラエラー</span>

エラーは`clarinet`関数がプライベートであるといっています。プライバシールールは、構造体、列挙型、関数、およびメソッドとモジュールに適用されます。

リスト7-8に示すように、定義の前に`pub`キーワードを追加することで`clarinet`関数をpublicにしましょう。

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod sound {
    pub mod instrument {
        pub fn clarinet() {
            // Function body code goes here
        }
    }
}

fn main() {
    // Absolute path
    crate::sound::instrument::clarinet();

    // Relative path
    sound::instrument::clarinet();
}
```

<span class="caption">リスト 7-8: `pub`キーワードを`mod instrument`と`fn clarinet`に追加すると`main`から関数を呼び出すことができます</span>

これでコンパイルできます。絶対パスと相対パスの両方を見てみましょう。なぜ`pub`キーワードを追加すると`main`でこれらのパスを使うのかを再確認します。

絶対パスの場合、クレートルートである`crate`で始まります。そこから、`sound`を持っており、クレートルートに定義されているモジュールです。 `sound`モジュールはpublicではありませんが、`main`関数は`sound`が定義されている同じモジュールで定義されているので、`main`から `sound`を参照することができます。次は`instrument`です。これは`pub`でマークされたモジュールです。`instrument`の親モジュールにアクセスできるので、`instrument`にアクセスすることができます。最後に`clarinet`は`pub`とマークされた関数であり、親モジュールにアクセスできるので、この関数呼び出しは機能します。

相対パスの場合、ロジックは最初のステップを除いて絶対パスと同じです。クレートルートから始めるのではなく、パスは`sound`から始まります。`sound`モジュールは`main`と同じモジュール内に定義されているので、`main`が定義されているモジュールからの相対パスが動作します。そして`instrument`と`clarinet`が`pub`でマークされているので、残りのパスが動作し、この関数呼び出しも有効です。

### `super 'による相対パスの開始

`super`で始まる相対パスを構築することもできます。そうすることは`..`でファイルシステムパスを開始するようなものです。パスは現在のモジュールではなく、*parent*モジュールから始まります。これはリスト7-9の例のように`clarinet`関数が`breathe_in`のパスを`super`で始まるように指定することで`breathe_in`関数を呼び出す場合に便利です。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
# fn main() {}
#
mod instrument {
    fn clarinet() {
        super::breathe_in();
    }
}

fn breathe_in() {
    // Function body code goes here
}
```

<span class="caption">リスト 7-9: 親モジュールを見るために`super`で始まる相対パスを使って関数を呼び出す</span>

`clarinet`関数は`instrument`モジュールに入っていますので`super`を使って`instrument`の親モジュールに行きます。この場合、ルートは `crate`です。そこから`breathe_in`を探して見つけます

`crate`で始まる絶対パスではなく`super`で始まる相対パスを選択したい理由は`super`を使うとコードを更新して別のモジュール階層にすることが容易になるからです。アイテムとアイテムを呼び出すコードが一緒に移動されます。たとえば`instrument`モジュールと`breathe_in`関数を`sound`という名前のモジュールに入れることにした場合、リスト7-10に示すように`sound`モジュールを追加するだけです。

<span class="filename">Filename: src/lib.rs</span>

```rust
mod sound {
    mod instrument {
        fn clarinet() {
            super::breathe_in();
        }
    }

    fn breathe_in() {
        // Function body code goes here
    }
}
```

<span class="caption">リスト 7-10: `sound`という名前の親モジュールを追加しても、相対パス`super::breathe_in`には影響しません</span>

`clarinet`関数からの`super::breathe_in`への呼び出しは、リスト7-9のようにパスを更新せずに引き続きリスト7-10で機能します。`clarinet`関数で`super::breathe_in`の代わりに`crate::breathe_in`を使用した場合、親の`sound`モジュールを追加するときに `clarinet`関数を更新して`crate::sound::breathe_in`の代わりにします。相対パスを使用すると、モジュールを並べ替えるときに必要な更新が少なくなる可能性があります。

### StructとEnumで `pub`を使う

モジュールと関数で示したのと同様の方法で、構造体とenumをpublicに指定することができます。詳細はいくつかあります。

struct定義の前に`pub`を使うと、構造体をpublicにします。ただし、構造体のフィールドはまだプライベートです。それぞれのフィールドを公開するかどうかは、ケースバイケースで選択することができます。リスト7-11では、パブリック`name`フィールドを持つpublic`plant::Vegetable`構造体を定義しましたが、プライベートの`id`フィールドを定義しました。

<span class="filename">Filename: src/main.rs</span>

```rust
mod plant {
    pub struct Vegetable {
        pub name: String,
        id: i32,
    }

    impl Vegetable {
        pub fn new(name: &str) -> Vegetable {
            Vegetable {
                name: String::from(name),
                id: 1,
            }
        }
    }
}

fn main() {
    let mut v = plant::Vegetable::new("squash");

    v.name = String::from("butternut squash");
    println!("{} are delicious", v.name);

    // The next line won't compile if we uncomment it:
    // println!("The ID is {}", v.id);
}
```

<span class="caption">リスト 7-11: いくつかのパブリックフィールドといくつかのプライベートフィールドを持つ構造体
</span>

`plant::Vegetable`構造体の`name`フィールドはpublicであるため、`main`ではドット表記を使って`name`フィールドに書き込んで読み込むことができます。プライベートであるため`main`の`id`フィールドは使用できません。`id`フィールドの値を表示している行のコメントを外して、どのようなエラーが発生しているのかを確認してください。また、`plant::Vegetable`はプライベートフィールドを持っているので、構造体は`Vegetable`のインスタンスを構築するパブリック関連関数を提供する必要があることに注意してください。`Vegetable`にそのような機能がなければ`main`に`Vegetable`のインスタンスを作成することはできません。なぜなら`main`のプライベート`id`フィールドの値を設定できないからです 。

対照的に、パブリックな列挙型を作成すると、そのバリエーションはすべて公開されます。リスト7-12に示すように`enum`キーワードの前に`pub`をつけることが必要です。

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod menu {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

fn main() {
    let order1 = menu::Appetizer::Soup;
    let order2 = menu::Appetizer::Salad;
}
```

<span class="caption">リスト 7-12: enumをパブリックとして指定すると、そのすべてのバリアントが公開されます。</span>

`Appetizer`enumを公開したので、`main`に`Soup`と`Salad`の亜種を使用することができました。

`pub`にはもう一つの状況があります。これは、最後のモジュールシステムの機能である`use`キーワードです。`use`を単独でカバーし`pub`と `use`をどのように組み合わせるかを見ていきましょう。

### スコープにパスを渡すための `use`キーワード

この章のリストにある関数を呼び出すために書いた多くのパスが長くて繰り返していると考えているかもしれません。たとえば、リスト7-8では、`clarinet`関数の絶対パスか相対パスかを選択したかどうかにかかわらず、`clarinet`を呼び出すたびに`sound`と`instrument`も指定しなければなりませんでした。しかし、パスを一度スコープに入れてから、そのパスのアイテムをローカルアイテムのように呼び出す方法があります。`use`キーワードです。リスト7-13では、`main`関数のスコープに`crate::sound::instrument`モジュールを持っています。 

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod sound {
    pub mod instrument {
        pub fn clarinet() {
            // Function body code goes here
        }
    }
}

use crate::sound::instrument;

fn main() {
    instrument::clarinet();
    instrument::clarinet();
    instrument::clarinet();
}
```

<span class="caption">リスト 7-13: モジュールを`use`と絶対パスでスコープに入れて、モジュール内の項目を呼び出すために指定しなければならないパスを短縮します</span>

スコープに`use`とパスを追加することは、ファイルシステムにシンボリックリンクを作成することに似ています。クレートルートに`use crate::sound::instrument`を追加することにより、`instrument`モジュールがクレートルートに定義されているかのように、`instrument`がその有効範囲内の有効な名前になりました。`instrument`モジュール内のアイテムは、より古い、完全なパスを通して到達することができます。あるいは、`use`で作成した新しい、より短いパスを通してアイテムに到達することができます。`use`でスコープに入れられたパスは、他のパスと同様にプライバシーをチェックします。

アイテムを`use`と相対パスでスコープに入れたいのであれば、相対パスを使ってアイテムを直接呼び出すのとは少し違いがあります。現在のスコープ内の名前から始めるのではなく、`use`と`self`を使います。リスト7-14は、絶対パスを使用したリスト7-13と同じ動作を得るために相対パスを指定する方法を示しています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod sound {
    pub mod instrument {
        pub fn clarinet() {
            // Function body code goes here
        }
    }
}

use self::sound::instrument;

fn main() {
    instrument::clarinet();
    instrument::clarinet();
    instrument::clarinet();
}
```

<span class="caption">リスト 7-14: `use`でモジュールをスコープに入れ、`self`で始まる相対パス</span>

`use`の後に指定されたときに`self`で相対パスを開始することは、将来不要になるかもしれません。それは人々が取り除くことに取り組んでいる言語の不一致です。

`use`で絶対パスを指定すると、アイテムを呼び出すコードがモジュールツリー内の別の場所に移動しても、アイテムを定義するコードが変更されたときとは異なり、更新が容易になります。たとえば、リスト7-13のコードを実行する場合、`main`関数の動作を`clarinet_trio`という関数に抽出し、その関数を`performance_group`という名前のモジュールに移動します。これはリスト7-15に示すように、`use`を変更する必要はありません。

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod sound {
    pub mod instrument {
        pub fn clarinet() {
            // Function body code goes here
        }
    }
}

mod performance_group {
    use crate::sound::instrument;

    pub fn clarinet_trio() {
        instrument::clarinet();
        instrument::clarinet();
        instrument::clarinet();
    }
}

fn main() {
    performance_group::clarinet_trio();
}
```

<span class="caption">リスト 7-15: 項目を呼び出すコードを移動するときに絶対パスを更新する必要はありません</span>

対照的に、相対パスを指定するリスト7-14のコードに同じ変更を加えた場合、 `use self::sound::instrument`を`use super::sound::instrument`に変更する必要があります。モジュールツリーが将来どのように変更されるかわからない場合、相対パスまたは絶対パスのどちらを更新するかを選択すると推測できますが、作成者は`crate`で始まる絶対パスを指定する傾向があります。リスト7-10で見たように、モジュールツリーの周りを互いに独立して移動するのではなく、モジュールツリーの周りを移動する可能性が高くなります。

### 他のアイテムと関数の慣用的な`use`パス

リスト7-13では、リスト7-16に示すコードではなく、`use crate::sound::instrument`を指定してから、`main`で`instrument::clarinet`と呼びだしたのはなぜでしょうか。

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod sound {
    pub mod instrument {
        pub fn clarinet() {
            // Function body code goes here
        }
    }
}

use crate::sound::instrument::clarinet;

fn main() {
    clarinet();
    clarinet();
    clarinet();
}
```

<span class="caption">リスト 7-16: `クラリネット`関数を`use`で呼び出し独自のユニークなものにする</span>

関数の場合、関数の親モジュールを`use`で指定し、その関数を呼び出すときに親モジュールを指定するのは慣用的です。リスト7-16のように、`use`で関数へのパスを指定するのではなく、関数がローカルに定義されていないことを確認しながら、フルパスの繰り返しを最小限に抑えます。

構造体、列挙型、およびその他の項目の場合、`use`で項目へのフルパスを指定することは慣用的です。例えば、リスト7-17は、標準ライブラリの`HashMap`構造体をスコープに持ち込む慣用的な方法を示しています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

<span class="caption">リスト 7-17: HashMapを慣用的な方法でスコープに持ち込む</span>

対照的にリスト7-18のコードでは、HashMapの親モジュールをスコープに入れても慣用的ではありません。このイディオムには強い理由はありません。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::collections;

fn main() {
    let mut map = collections::HashMap::new();
    map.insert(1, 2);
}
```

<span class="caption">リスト 7-18: 独自の方法で `HashMap`をスコープに持ち込む</span>

このイディオムの例外は、`use`ステートメントが同名の2つの項目をスコープに持ち込むことができない場合です。リスト7-19は、異なる親モジュールを持つ2つの`Result`型をスコープに持ち込んで参照する方法を示しています。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
#     Ok(())
}
fn function2() -> io::Result<()> {
#     Ok(())
}
```

<span class="caption">リスト 7-19: 同じ名前の2つの型を同じスコープに組み込むには、親モジュールを使用する必要があります</span>

代わりに、`use std::fmt::Result`と`use std::io::Result`を指定した場合、同じスコープ内に2つの`Result`型があり、`Result`を使いました。それを試してみて、どのコンパイラエラーが発生したかを見てください。

### `as`キーワードで型名の名前を変更する

同じ名前の2つの型を同じスコープに持たせる問題のもう1つの解決方法があります。型の新しいローカル名を`use`の後に`as`を追加し、新しい名前を指定することで指定できます。リスト7-20は、リスト7-19のコードを`as`を使って2つの`Result`型の名前を変更して書くもう1つの方法を示しています。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
#     Ok(())
}
fn function2() -> IoResult<()> {
#     Ok(())
}
```

<span class="caption">Listing 7-20: 型が`as`キーワードで有効範囲に入ったときに型の名前を変更する</span>

2番目の`use`ステートメントでは、`std::io: Result`型の新しい名前`IoResult`を選択しました。これは、`std::fmt`の`Result`と競合しません またスコープに持ち込まれる。これはまた慣用的であると考えられている。コードリスト7-19とリスト7-20のコードの選択は自分次第です。

### `pub use`で名前を再エクスポートする

`use`キーワードでスコープに名前を付けると、新しいスコープで使用できる名前はprivateになります。コードを呼び出すコードが、同じようにその型で定義されているかのように型を参照できるようにしたい場合は、`pub`と`use`を組み合わせることができます。このテクニックは、アイテムを範囲に入れているだけでなく、そのアイテムを他の人が自分のスコープに持ち込むことができるようにするために*再エクスポート*と呼ばれます。

たとえば、リスト7-21は、リスト7-15のコードを示し、`performance_group`モジュール内の`use`が pub use`に変更されています。

<span class="filename">ファイル名: src/main.rs</span>

```rust
mod sound {
    pub mod instrument {
        pub fn clarinet() {
            // Function body code goes here
        }
    }
}

mod performance_group {
    pub use crate::sound::instrument;

    pub fn clarinet_trio() {
        instrument::clarinet();
        instrument::clarinet();
        instrument::clarinet();
    }
}

fn main() {
    performance_group::clarinet_trio();
    performance_group::instrument::clarinet();
}
```

<span class="caption">リスト 7-21: `pub use`で新しいスコープから使用するコードに名前を付ける</span>

`pub use`を使うことで、`main`関数は`performance_group::instrument::clarinet`でこの新しいパスを通して`clarinet`関数を呼び出すことができます。`pub use`を指定しなかった場合、`clarinet_trio`関数はその範囲で`instrument::clarinet`を呼び出すことができますが、`main`はこの新しいパスを利用することができません。

### 外部パッケージの使用

第2章では、数当てゲームをプログラミングしました。このプロジェクトでは、乱数を得るために外部パッケージ`rand`を使いました。プロジェクトで`rand`を使うために、この行を*Cargo.toml*に追加しました。

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[dependencies]
rand = "0.5.5"
```

*Cargo.toml*の依存関係として`rand`を追加すると、Cargoは`rand`パッケージとその依存関係を*https://crates.io*からダウンロードし、そのコードをプロジェクトに利用できるようにします。

次に、`rand`定義をパッケージのスコープに持たせるために、パッケージの名前`rand`で始まる`use`行を追加し、スコープに入れたい項目をリストアップしました。第2章の「乱数の生成」の節では、`Rng`トレイトをスコープに持ち込み、`rand::thread_rng`関数を呼び出したことを思い出してください。

```rust,ignore
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1, 101);
}
```

コミュニティのメンバーが*https://crates.io*で公開しているパッケージがたくさんあり、そのいずれかをパッケージに引っ張るのは、パッケージの*Cargo.toml*にリストして項目を定義するために、それらを`use`でパッケージのスコープに入れます。

標準ライブラリ（`std`）もパッケージの外部にあるクレートです。標準ライブラリにはRust言語が付属しているので、*Cargo.toml*を `std`をインクルードするように変更する必要はありませんが、`use`で参照して標準ライブラリが定義した項目をパッケージのスコープに持ち込み、`HashMap`のように書くこともできます。

```rust
use std::collections::HashMap;
```

これは標準ライブラリクレートの名前である`std`で始まる絶対パスです。

### 大規模な`use`リストを整理するためのネストされたパス

同じパッケージまたは同じモジュールで定義された多数の項目を使用する場合、各項目を独自の行にリストすると、ファイル内に多くの縦方向のスペースが必要になります。たとえば、リスト2-4の数あてゲームで使用したこれら2つの`use`ステートメントは、`std`の項目をスコープに持ち込みます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::cmp::Ordering;
use std::io;
// ---snip---
```

リスト7-22のようにネストしたパスを使用して、パスの共通部分を指定し、次に2つのコロンを指定し、異なるパスの部分のリストを中括弧で囲むことで、同じ項目を2つではなく1つの行に1つのスコープにすることができます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
use std::{cmp::Ordering, io};
// ---snip---
```

<span class="caption">リスト 7-22:ネストされたパスを指定して、同じプレフィックスを持つ複数の項目を2つではなく1行にスコープに入れる</span>

同じパッケージやモジュールから多くの項目をスコープに入れたプログラムでは、ネストされたパスを使うと、たくさんの`use`ステートメントの数を減らすことができます。

また、あるパスが完全に共有されているパスを別のパスの一部と重複排除することもできます。たとえば、リスト7-23は2つの`use`文を示しています。一つは`std::io`をスコープに持ち、もう一つは `std::io::Write`をスコープに持ちます。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
use std::io;
use std::io::Write;
```

<span class="caption">リスト 7-23: 2つのパスを2つの`use`ステートメントのスコープに入れます。1つはもう一方のサブパスです</span>

これら2つのパスの共通部分は`std::io`であり、これが完全な最初のパスです。これらの2つのパスを1つの`use`ステートメントに重複排除するには、リスト7-24に示すように、ネストされたパスに`self`を使用できます。

<span class="filename">ファイル名: src/lib.rs</span>

```rust
use std::io::{self, Write};
```

<span class="caption">リスト 7-24: リスト7-23のパスを1つの `use`文に重複除外します</span>

これは `std::io`と` std::io::Write`の両方をスコープに持ち込みます。

### すべてのPublic定義をGlob演算子でスコープに変換する

パスに定義された*すべて*のパブリックアイテムをスコープに持たせたい場合は、そのパスの後にglob演算子`*`を指定することができます。

```rust
use std::collections::*;
```

この`use`ステートメントは、`std::collections`で定義されたすべてのパブリックアイテムを現在のスコープに持ち込みます。

glob演算子の使用には注意してください。どの名前がスコープにあり、どの名前がプログラムで使用されているかを知るのが難しくなります。

glob演算子は、テスト中の全てを`tests`モジュールに持っていくためにテストするときによく使われます。第11章の「テストの書き方」の節でそれについて話します。glob演算子は、初期化処理の一部として使用されることもあります。そのパターンの詳細については、[標準ライブラリのドキュメント](../../ std / prelude / index.html＃other-preludes)を参照してください。

### モジュールを別々のファイルに分ける

ここまでの例では、1つのファイルに複数のモジュールを定義していました。モジュールが大きくなると、その定義を別のファイルに移動して、コードをナビゲートしやすくなります。

例えば、リスト7-8のコードから始めれば、クレートルートファイル(この場合は*src/main.rc*)を変更して、リストに示すコードを含むように、`sound`モジュールを独自のファイル*src/sound.rs*に移動できます

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
mod sound;

fn main() {
    // 絶対パス
    crate::sound::instrument::clarinet();

    // 相対パス
    sound::instrument::clarinet();
}
```

<span class="caption">リスト 7-25: 本体が*src/sound.rs*にある`sound`モジュールを宣言します。</span>

*src/sound.rs*は、リスト7-26に示す`sound`モジュールの本体から定義を取得します。

<span class="filename">ファイル名: src/sound.rs</span>

```rust
pub mod instrument {
    pub fn clarinet() {
        // Function body code goes here
    }
}
```

<span class="caption">リスト 7-26: *src/sound.rs*の`sound`モジュールの中の定義</span>

ブロックの代わりに`mod sound`の後にセミコロンを使うと、モジュールの内容をモジュールと同じ名前の別のファイルからロードするようにRustに指示します。

この例を続行し、`instrument`モジュールをそれ自身のファイルに抽出するために、`instrument`モジュールの宣言だけを含むように*src/sound.rs*を変更します。

<span class="filename">ファイル名: src/sound.rs</span>

```rust
pub mod instrument;
```

次に、`instrument`モジュールで定義された定義を格納する*src/sound*ディレクトリとファイル*src/sound/instrument.rs*を作成します。

<span class="filename">ファイル名: src/sound/instrument.rs</span>

```rust
pub fn clarinet() {
    // Function body code goes here
}
```

モジュールツリーは同じままであり、定義が異なるファイルに存在していても、`main`の関数呼び出しは何の変更もなしに機能し続けます。これにより、モジュールのサイズが大きくなるにつれてモジュールを新しいファイルに移動できます。

## まとめ

Rustは、パッケージをクレートに編成し、モジュールをクレートにモジュール化し、絶対パスまたは相対パスを指定することによって、あるモジュールで定義されたアイテムを別のモジュールから参照する方法を提供します。これらのパスは`use`ステートメントでスコープに入れられ、そのスコープ内のアイテムの複数の使用に短いパスを使用できます。モジュールはデフォルトで非公開のコードを定義しますが、`pub`キーワードを追加することで定義を公開することもできます。

次に、標準ライブラリのコレクションデータ構造を見ていきます。これは、きちんとしたきれいなコードで使用できます。