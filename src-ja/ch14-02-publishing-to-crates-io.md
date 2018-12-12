## Crates.ioにクレートを公開する

私たちはプロジェクトの依存関係として[crates.io](https://crates.io)<!--ignore-->のパッケージを使用しましたが、独自のパッケージを公開して他の人とコードを共有することもできます。[crates.io](https://crates.io)のcrateレジストリは、パッケージのソースコードを配布するので、主にオープンソースのコードをホストします。

RustとCargoには、公開されたパッケージを人々が使いやすく見つけやすくするための機能があります。これらの機能のいくつかについて次に説明し、次にパッケージを公開する方法を説明します。

### 役に立つドキュメンテーションコメントを行う

パッケージを正確にドキュメント化することで、他のユーザーがいつ、どのように使用するのかを知ることができますので、時間をかけてドキュメントを書く価値があります。第3章では、2つのスラッシュ`//`を使って錆コードをコメントする方法について説明しました。Rustには、HTMLドキュメントを生成する*ドキュメンテーションコメント*として知られている、ドキュメントのための特定の種類のコメントもあります。HTMLには、レートの実装方法とは対照的に、クレートを使用する方法を知りたいプログラマー向けのパブリックAPI項目のドキュメントコメントの内容が表示されます。

ドキュメントコメントでは、2つではなく、3つのスラッシュ`///`を使用し、テキストの書式設定のためのMarkdown記法をサポートしています。ドキュメント化している項目の直前にドキュメンテーションコメントを置いてください。リスト14-1は、`my_crate`という名前のクレート内の`add_one`関数のドキュメンテーションコメントを示しています。

<span class="filename">ファイル名: src/lib.rs</span>

```rust,ignore
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, my_crate::add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

<span class="caption">リスト 14-1: 関数のドキュメンテーションコメント</span>

ここでは、`add_one`関数が何をするのかを説明し、`Examples`という見出しでセクションを開始し、`add_one`関数の使い方を示すコードを提供します。`cargo doc`を実行することによって、このドキュメンテーションコメントからHTML文書を生成することができます。このコマンドはRustと一緒に配布された`rustdoc`ツールを実行し、生成されたHTMLドキュメントを*target/doc*ディレクトリに置きます。

利便性のために、`cargo doc --open`を実行すると、現在のクレートのドキュメント（クレートの依存関係のすべてのドキュメント）のHTMLがビルドされ、Webブラウザで結果が開きます。`add_one`関数に移動すると、図14-1に示すように、ドキュメンテーションコメントのテキストがどのように表示されるのかがわかります：

<img alt="add_one関数のHTMLドキュメント" src="img/trpl14-01.png" class="center" />

<span class="caption">図 14-1: `add_one`関数のHTMLドキュメント</span>

#### よく使われるセクション

リスト14-1の `#Examples`マークダウン見出しを使って、HTMLに"Examples"というタイトルのセクションを作成しました。ここでは、よく使用するセクションをいくつか紹介します。

* **Panics**: ドキュメント化されている機能がパニックに陥る可能性のあるシナリオ。プログラムのパニックを望まない機能の呼び出し側は、これらの状況で関数を呼び出さないようにする必要があります。
* **Errors**:関数が結果を返す場合、発生する可能性があるエラーの種類とその条件が返される条件を記述することで、さまざまな種類のエラーを処理するコードをさまざまな方法で記述できるようになります。
* **Safety**: 関数が呼び出すのに`unsafe`(`unsafe`については第19章で議論します)なら、関数が安全でない理由を説明し、関数が呼び出し側が維持することを期待する不変量をカバーするセクションが存在するはずです。

ほとんどのドキュメンテーションコメントはこれらのセクションのすべてを必要としませんが、コードを呼び出す人々が知りたいと思うコードの面を思い出させる良いチェックリストです。

#### テストとしてのドキュメンテーションコメント

ドキュメンテーションコメントにサンプルコードブロックを追加すると、ライブラリの使い方を示すのに役立ちます。そうすることで追加のメリットが得られます。`cargo test`を実行すると、ドキュメンテーションのコード例がテストとして実行されます。例を持つドキュメントより優れているものはありません。しかし、ドキュメントが書かれてからコードが変更されたため、動作しない例よりも悪いものはありません。リスト14-1の`add_one`関数のドキュメンテーションで`cargo test`を実行すると、次のようなテスト結果のセクションが表示されます。

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

さて、例の`assert_eq!`がパニックするように、関数か例を変更し、再度`cargo test`を実行したら、docテストが、例とコードがお互いに同期されていないことを捕捉するところを目撃するでしょう。

#### 含まれている要素にコメントする

ドキュメンテーションコメントの別のスタイル`//!`は、コメントに続く項目にドキュメントを追加するのではなく、コメントを含む項目にドキュメントを追加します。通常、クレートルートファイル（慣習的に*src/lib.rs*）内またはこれらのドキュメントのコメントを使用して、クレートまたはモジュール全体をドキュメント化します。

たとえば、`add_one`関数を含む`my_crate`クレートの目的を説明するドキュメントを追加したい場合は、リスト14-2のように`//!`で始まるドキュメンテーションコメントを*src/lib.rs*ファイルの先頭につけることができます。

```rust,ignore
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

<span class="caption">リスト 14-2: 全体として`my_crate`クレートにドキュメントをつける</span>

`//!`で始まる最後の行の後にコードがないことに注目してください。`///`ではなく`//!`でコメントを開始したので、このコメントの後に続く項目ではなく、このコメントを含む項目をドキュメント化しています。この場合、このコメントを含む項目はクレートルートである*src/lib.rs*ファイルです。これらのコメントはクレート全体を解説します。

`cargo doc --open`を実行すると、これらのコメントは、図14-2に示すように、クレート内のパブリックアイテムのリストの上にある`my_crate`のドキュメントの先頭ページに表示されます。

<img alt="クレート全体を解説するコメントを含むmy_crateの描画されたドキュメンテーション" src="img/trpl14-02.png" class="center" />

<span class="caption">図 14-2: クレート全体を解説するコメントを含む`my_crate`の描画されたドキュメンテーション</span>

アイテム内のドキュメンテーションコメントは、特にクレートやモジュールの説明に役立ちます。これらを使用して、コンテナの全体的な目的を説明し、ユーザーがクレートの組織を理解するのを助けます。

### `pub use`で便利な公開APIをエクスポートする

第7章では、`mod`キーワードを使用してコードをモジュールに編成する方法、`pub`キーワードを使用してアイテムをパブリックにする方法、`use`キーワードを使用してアイテムをスコープに持ち込む方法について説明しました。しかし、クレートを開発している間、自分にとって意味のある構造は、ユーザにとってはあまり便利でないかもしれません。構造体を複数のレベルを含む階層に編成することもできますが、階層の中で深く定義した型を使用したい人は、その型が存在するかどうかを調べるのが難しいかもしれません。そのような人は`use my_crate::UsefulType`の代わりに`use my_crate::some_module::another_module::UsefulType;`と入力するのを煩わしく感じる可能性もあります。

パブリックAPIの構造は、クレートを公開する際の主な考慮事項です。クレートを使用する人はあなたのものより構造にあまり精通しておらず、クレートが大きなモジュール階層を持っている場合、使用したい部分を見つけるのが難しいかもしれません。

良いことは、構造が他人が他のライブラリから使用するのに便利ではない場合、内部的な体系を再構築する必要はないということです。代わりに、要素を再エクスポートし、`pub use`で自分の非公開構造とは異なる公開構造にできます。再エクスポートは、ある場所の公開要素を一つ取り、別の場所で定義されているかのように別の場所で公開します。

たとえば、美術の概念をモデル化するために「アート」という名前のライブラリを作成したとします。 このライブラリには、リスト14-3に示すように、 `PrimaryColor`と` SecondaryColor`という名前の2つの列挙型と `mix`という名前の関数を含む` utils`モジュールを含む `kinds`モジュールの2つのモジュールがあります：

<span class="filename">ファイル名: src/lib.rs</span>

```rust,ignore
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

<span class="caption">リスト 14-3:アイテムを`kind`と`utils`モジュールに編成した `art`ライブラリ</span>

図14-3は、`cargo doc`によって生成されたこのクレートのドキュメントの最初のページを示しています。

<img alt="`kinds`と`utils`モジュールを列挙する`art`のドキュメンテーションのトップページ" src="img/trpl14-03.png" class="center" />

<span class="caption">図 14-3: `kinds`と`utils`モジュールを列挙する`art`のドキュメンテーションのトップページ</span>

`PrimaryColor`も`SecondaryColor`型も、`mix`関数もトップページには列挙されていないことに注意してください。`kinds`と`utils`をクリックしなければ、参照することができません。

このライブラリに依存する別のクレートには、現在定義されているモジュール構造を指定して`art`からの項目をスコープに持ってくる`use`ステートメントが必要です。リスト14-4は、`art`クレートの`PrimaryColor`と`mix`アイテムを使用するクレートの例を示しています。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

<span class="caption">リスト 14-4: 内部構造もエクスポートされてartクレートの要素を使用するクレート</span>

リスト14-4のコードの作者は、`art`クレートを使って、`PrimaryColor`が`kinds`モジュールにあり、`mix`が`utils`モジュールにあることを理解しなければなりませんでした。`art`クレートのモジュール構造は、 `art`クレートの使用者よりも、`art`クレートに取り組む開発者などに関係が深いです。クレートの一部を`kind`モジュールと`utils`モジュールに編成する内部構造は、`art`クレートの使い方を理解しようとする人にとって有益な情報を含んでいません。代わりに、`art`クレートのモジュール構造は混乱を招きます。なぜなら、開発者はどこを見るべきかを把握しなければならず、`use`ステートメントでモジュール名を指定する必要があるからです。

公開APIから内部構造を削除するには、リスト14-3の`art`クレートのコードを変更して、リスト14-5に示すように`pub use`ステートメントを追加してアイテムをトップレベルで再エクスポートすることができます。

<span class="filename">ファイル名: src/lib.rs</span>

```rust,ignore
//! # Art
//!
//! A library for modeling artistic concepts.

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

<span class="caption">リスト 14-5: pub use文を追加して要素を再エクスポートする</span>

`cargo doc`がこのクレート用に生成するAPIドキュメントは、図14-4に示すように、フロントページに再エクスポートをリストしてリンクするようになり、`PrimaryColor`と`SecondaryColor`タイプと`mix`関数を簡単にします 見つけるには。

<img alt="再エクスポートを列挙する`art`のドキュメンテーションのトップページ" src="img/trpl14-04.png" class="center" />

<span class="caption">図 14-4: 再エクスポートを列挙する`art`のドキュメンテーションのトップページ</span>
 
`art`クレートのユーザは、リスト14-4に示すように内部構造を表示したり使用したりすることができます（リスト14-6を参照）。

<span class="filename">ファイル名: src/main.rs</span>

```rust,ignore
use art::PrimaryColor;
use art::mix;

fn main() {
    // --snip--
}
```

<span class="caption">リスト 14-6: `art`クレートの再エクスポートされた要素を使用するプログラム/span>

入れ子になったモジュールがたくさんある場合は、`pub use`でトップレベルのタイプを再エクスポートすると、クレートを使用する人の経験に大きな違いが生じます。

便利な公開API構造を作成することは、科学よりも芸術の領域であり、ユーザーにとって最適なAPIを見つけることができます。`pub use`を選択すると、クレートを内部的に構造化する方法に柔軟性がもたらされ、その内部構造をあなたのユーザに提示するものから切り離すことができます。インストールしたクレートのコードの一部を見て、内部構造が公開APIと異なるかどうかを確認してください。

### Crates.ioのアカウントをセットアップする

任意のクレートを公開する前に、[crates.io](https://crates.io)<!--ignore-->にアカウントを作成し、APIトークンを取得する必要があります。 これを行うには、ホームページの[crates.io](https://crates.io)<!--ignore-->にアクセスし、GitHubアカウントからログインしてください。 （GitHubアカウントは現在のところ必要ですが、将来アカウントを作成する他の方法をサポートしているかもしれません）。ログインしたら、[https://crates.io/me/](https://crates.io/me/)<!--ignore-->のアカウント設定にアクセスしてくださいあなたのAPIキーを取得します。次に、APIキーを使って`cargo login`コマンドを実行します。

```text
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

このコマンドはCargoにAPIトークンを通知し、*~/.cargo/credentials*にローカルに格納します。このトークンは*シークレット*であることに注意してください。他の誰とも共有しないでください。何らかの理由で他の人と共有している場合は、それを取り消して[crates.io](https://crates.io)<!-- ignore -->に新しいトークンを生成する必要があります。

### 新しいクレートにメタデータを追加する

アカウントはできたので、公開したいクレートを持っているとしましょう。公開する前に、クレートの*Cargo.toml*ファイルの`[package]`セクションに追加することによって、クレートにメタデータを追加する必要があります。

クレートには、独自の名前が必要でしょう。クレートをローカルで作成している間、クレートの名前はなんでもいい状態でした。 ただし、[crates.io](https://crates.io)<!--ignore-->のクレート名は、先着順に割り当てられます。いったんクレートの名前が取られてしまうと、誰もその名前のクレートを公開することはできません。サイトで使用する名前を検索し、使用されているかどうかを調べます。もしそうでなければ、`[package]`の*Cargo.toml*ファイルにある名前を編集して公開用の名前を使用します。

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

一意の名前を選択したとしても、この時点で`cargo publish`を実行してクレートを公開すると、警告が表示され、エラーが表示されます。

```text
$ cargo publish
    Updating registry `https://github.com/rust-lang/crates.io-index`
warning: manifest has no description, license, license-file, documentation,
homepage or repository.
--snip--
error: api errors: missing or empty metadata fields: description, license.
```

その理由は、いくつかの重要な情報が欠落しているということです。クレートが何をしているのか、どのような条件で使用できるのかを人々が知るための説明とライセンスが必要です。このエラーを解決するには、この情報を*Cargo.toml*ファイルに含める必要があります。

クレートと一緒に検索結果に表示されるため、1つまたは2つの説明だけを追加します。`license`フィールドには、*ライセンス識別子の値*を与える必要があります。[Linux Foundationのソフトウェアパッケージデータ交換(SPDX)][spdx]には、この値に使用できる識別子がリストされています。たとえば、MITライセンスを使用してあなたの箱をライセンスしたことを指定するには、`MIT`識別子を追加します。

[spdx]: http://spdx.org/licenses/

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

SPDXに表示されないライセンスを使用する場合は、そのライセンスのテキストをファイルに入れ、プロジェクトにファイルを組み込み、`license`キーを使用する代わりに、`license-file`を使用してその名前を指定する必要があります。

どのライセンスがプロジェクトに適しているかについてのガイダンスは、この本の範囲を超えています。Rustコミュニティの多くの人々は、`MIT OR Apache-2.0`の二重ライセンスを使って、Rustと同じ方法でプロジェクトのライセンスを取得しています。このプラクティスでは、複数のライセンスIDを指定して複数のライセンスを所有することもできます。

独自の名前、バージョン、クレート作成時に`cargo new`が追加した筆者の詳細、説明、ライセンスが追加され、公開準備のできたプロジェクト用の*Cargo.toml*ファイルは以下のような見た目になっていることでしょう:

<span class="filename">ファイル名: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```
[Cargo's documentation](https://doc.rust-lang.org/cargo/)では、他の人がクレートを見つけやすく使いやすくするために指定できるメタデータについて説明しています。

### Crates.ioに公開する

アカウントを作成し、APIトークンを保存し、クレートの名前を選択し、必要なメタデータを指定したら、すぐに公開できます。クレートを公開すると[crates.io](https://crates.io)<!--ignore-->に特定のバージョンがアップロードされます。

公開は*永久*であるため、クレートを公開するときは注意してください。バージョンを上書きすることはできず、コードを削除することはできません。[crates.io](https://crates.io)<!--ignore-->の主要な目標の1つは、コードの永続的なアーカイブとして機能することで、[crates.io](https://crates.io)<!--ignore-->のボックスに依存するすべてのプロジェクトのビルドは引き続き動作します。バージョンの削除を許可すると、その目標を達成することは不可能になります。ただし、公開できるクレートバージョンの数に制限はありません。

`cargo publish`コマンドを再度実行してください。今度は成功します。

```text
$ cargo publish
 Updating registry `https://github.com/rust-lang/crates.io-index`
Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
 Finished dev [unoptimized + debuginfo] target(s) in 0.19 secs
Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Rustコミュニティとコードを共有しました。誰でもプロジェクトの依存としてクレートを簡単に追加できます。

### 既存のクレートの新バージョンを公開する

クレートを変更して新しいバージョンをリリースする準備が整ったら、*Cargo.toml*ファイルで指定された`version`値を変更して再公開します。[セマンティックバージョンルール][semver]を使って、あなたが行った変更の種類に基づいて適切な次のバージョン番号が何であるかを決定します。次に、新しいバージョンをアップロードするために`cargo publish`を実行します。

[semver]: http://semver.org/

### `cargo yank`でCrates.ioからバージョンを削除する

以前のバージョンのクレートを削除することはできませんが、今後のプロジェクトでは新しい依存関係として追加することはできません。これは、クレートバージョンが何らかの理由で壊れている場合に便利です。このような状況では、Cargoはクレートバージョンの取り下げをサポートしています。

バージョンを取り下げると、他の既存のプロジェクトには、引き続きダウンロードし、そのバージョンに依存させ続けつつ、 新規プロジェクトが新しくそのバージョンに依存しだすことを防ぎます。本質的に、取り下げは、*Cargo.lock*を持つすべてのプロジェクトが中断しないことを意味し、生成される*Cargo.lock*ファイルはヤンクされたバージョンを使用しません。

あるバージョンのクレートを取り下げるには、`cargo yank`を実行し、取り下げたいバージョンを指定します。

```text
$ cargo yank --vers 1.0.1
```

コマンドに `--undo`を追加することで、取り下げを元に戻して、バージョンに応じてプロジェクトを再開することもできます。

```text
$ cargo yank --vers 1.0.1 --undo
```

*取り下げ*はコードを削除しません。たとえば、誤ってアップロードされた秘密鍵を削除するための機能ではありません。そのような場合は、すぐにそれらの秘密鍵をリセットする必要があります。
