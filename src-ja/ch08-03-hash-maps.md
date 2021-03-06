## 関連付けられた値を持つキーをhash mapに格納する

一般的なコレクションの最後は*hash map*です。`HashMap<K、V`>型は、型`K`のキーを型`V`の値にマッピングします。これらのキーと値をどのようにメモリに格納するかを決める*ハッシュ関数*を使ってこれを行います。多くのプログラミング言語はこの種のデータ構造をサポートしていますが、ハッシュ、マップ、オブジェクト、ハッシュテーブル、ディクショナリ、連想配列などの異なる名前を使用することがよくあります。

hash mapは、vectrを使用する場合と同じように、インデックスを使用せずに任意のタイプのキーを使用してデータをルックアップする場合に便利です。 たとえば、ゲームでは、各キーがチームの名前で値が各チームのスコアであるhash mapで各チームのスコアを追跡できます。チーム名を指定するとスコアを取得できます。

このセクションでは、hash mapの基本的なAPIについて説明しますが、標準ライブラリでは`HashMap<K、V>`で定義されている関数にはもっと多くの機能が隠されています。詳細は標準ライブラリのマニュアルを参照してください。

### 新しいhash mapの作成


`new`で空のhash mapを作成し、`insert`で要素を追加することができます。リスト8-20では、名前がBlueとYellowの2つのチームの得点を記録しています。ブルーチームは10ポイントでスタートし、イエローチームは50ポイントでスタートします。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

<span class="caption">リスト 8-20:新しいhash mapを作成し、いくつかのキーと値を挿入する</span>

標準ライブラリのコレクション部分から`HashMap`を最初に`使用する`必要があることに注意してください。3つの一般的なコレクションのうち、これは最も頻繁に使用されるものではないため、プレリュードに自動的に適用される機能には含まれていません。 hash map、標準ライブラリからのサポートも少なくなっています。例えば、それらを構築するための組み込みマクロはありません。

vectorと同様に、hash mapはヒープ上にデータを格納します。この`HashMap`は`String`型のキーと`i32`型の値を持っています。 vectorと同様に、hash mapはすべてのキーは同じタイプでなければならず、すべての値は同じタイプでなければなりません。

hash mapを構築する別の方法は、タプルのベクトルに対して`collect`メソッドを使うことです。ここで、各タプルはキーとその値から成ります。`collect`メソッドは`HashMap`を含む多くのコレクション型にデータを集めます。たとえば、チーム名と初期スコアが2つの別々のベクトルであれば、`zip`メソッドを使用して"Blue"が10とペアになっているタプルのベクトルを作成することができます。リスト8-21に示すように、`collect`メソッドを使ってタプルのベクトルをhash mapにすることができます。

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

<span class="caption">リスト 8-21: チームのリストと得点のリストからhash mapを作成する</span>

`HashMap<_, _>`型の注釈はここで必要です。なぜなら、指定しない限り何を望んでいるのかわからないため、さまざまなデータ構造に`collect`することができるからです。ただし、キーと値の型の引数では、アンダースコアを使用し、hash mapに含まれるデータの型に基づいて型を推測できます。

### hash mapと所有権

`Copy`特性を実装する型の場合、`i32`のように、値はhash mapにコピーされます。リスト8-22に示すように、`String`のような所有値の場合、値は移動され、hash mapはそれらの値の所有者になります。

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point, try using them and
// see what compiler error you get!
```

<span class="caption">リスト 8-22: hash mapが挿入されると、そのキーと値が所有されていることを示す</span>

変数`field_name`と`field_value`を、`insert`を呼び出してhash mapに移動した後で使用することはできません。

値への参照をhash mapに挿入すると、値はhash mapに移動されません。 参照が指す値は、少なくともhash mapが有効である限り有効でなければなりません。これらの問題については、第10章の「ライフタイムを使用した参照の検証」のセクションで詳しく説明します。

### hash mapの値へのアクセス

リスト8-23に示すように、キーを`get`メソッドに渡すことで、hash mapから値を取得できます。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

<span class="caption">リスト 8-23: hash mapに保存されているBlueチームのスコアにアクセスする</span>

ここでは、`スコア`はBlueチームに関連付けられた値を持ち、結果は`Some(&10)`になります。`get`は`Option<&V>`を返すので、結果は`Some`でラップされます。そのキーの値がhash mapにない場合、`get`は`None`を返します。プログラムは、第6章で取り上げた方法の1つで、`Option`を処理する必要があります。

`for`ループを使って、vectorと同様の方法でhash mapの各キー/値の組を繰り返し処理できます。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

このコードは各ペアを任意の順序で出力します。

```text
Yellow: 50
Blue: 10
```

### hash mapの更新

キーと値の数は増えますが、各キーは一度に1つの値しか関連付けられません。hash mapのデータを変更する場合は、キーにすでに値が割り当てられている場合に、そのケースの処理方法を決定する必要があります。古い値を新しい値に置き換えて、古い値を完全に無視することができます。古い値を保持して新しい値を無視し、キーにすでに値が*ない場合*にのみ新しい値を追加することができます。または、古い値と新しい値を組み合わせることもできます。これらのそれぞれを行う方法を見てみましょう。

#### 値を上書きする

hash mapにキーと値を挿入し、同じキーに別の値を挿入すると、そのキーに関連付けられた値が置き換えられます。リスト8-24のコードでは`insert`を2回呼び出していますが、Blueチームのキーの値を両方とも挿入するため、hash mapには1つのキーと値のペアしか含まれません。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

<span class="caption">リスト 8-24:特定のキーに格納されている値を置き換える</span>

このコードは`{"Blue": 25}`を出力します。`10`の元の値は上書きされています。

#### キーに値がない場合にのみ値を挿入する

特定のキーに値があるかどうかをチェックし、キーに値がない場合は値を挿入するのが一般的です。hash mapには、これと呼ばれる`entry`という特別なAPIがあります。このAPIは、チェックしたいキーを引数として受け取ります。`entry`関数の戻り値は、存在するかもしれないし、存在しないかもしれない値を表す`Entry`と呼ばれるenumです。Yellowチームのキーに関連する値があるかどうかを確認したいとします。そうでない場合は、値50を挿入し、Blueチームに同じ値を挿入します。`entry` APIを使用すると、コードはリスト8-25のようになります。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

<span class="caption">リスト 8-25: キーがまだ値を持っていない場合にのみ挿入する`entry`メソッドの使用
</span>

`Entry`の`or_insert`メソッドは、対応する`Entry`キーが存在すればその値への変更可能な参照を返すように定義され、そうでなければこのキーの新しい値として引数を挿入し、変更可能な参照を返します。このテクニックは、ロジックを書くよりもはるかにクリーンで、さらに、借用チェッカーでうまく機能します。

リスト8-25のコードを実行すると`{"Yellow": 50, "Blue": 10}`が出力されます。黄色のチームにはすでに値がないので、`entry`への最初の呼び出しは、値が`50`のYellowチームのキーを挿入します。ブルーチームが既に値`10`を持っているので、 `entry`への2回目の呼び出しはhash mapを変更しません。

#### 古い値に基づく値の更新

hash mapの別の一般的な使用例は、キーの値をルックアップし、古い値に基づいてキーの値を更新することです。たとえば、リスト8-26は、各単語がテキストに何回出現するかを数えるコードを示しています。単語をキーとしてhash mapを使用し、その単語を何回見たかを追跡するために値を増やします。単語を見たのが初めての場合は、最初に値`0`を挿入します。

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

<span class="caption">リスト 8-26: 単語とカウントを格納するhash mapを使って単語の出現を数える</span>

このコードは `{"world": 2、"hello": 1, "wonderful": 1}`を出力します。`or_insert`メソッドは実際にこのキーの値に変更可能な参照（`&mut V`）を返します。ここでは、その可変参照を`count`変数に格納しています。その値に代入するには、まずアスタリスク（`*`）を使って`count`を参照解除する必要があります。変更可能な参照は`for`ループの終わりで範囲外になるので、これらの変更はすべて安全であり、借用ルールによって許可されます。

### ハッシュ関数

デフォルトでは、`HashMap`はサービス拒否（DoS）攻撃に抵抗することができる"暗号的に強力"[^siphash]なハッシュ関数を使用します。 利用可能な最速のハッシングアルゴリズムではありませんが、パフォーマンスの低下に伴うより良いセキュリティのトレードオフが価値があります。コードをプロファイリングして、デフォルトのハッシュ関数が遅すぎると分かったら、別の*hasher*を指定して別の関数に切り替えることができます。hasherは`BuildHasher`特性を実装する型です。第10章では、特性とその実装方法について説明します。必ずしも独自のhasherをゼロから実装する必要はありません。[crates.io]（https://crates.io）には、多くの一般的なハッシュアルゴリズムを実装しているハッシャーを提供する他のRustユーザーが共有するライブラリがあります。

[^siphash]: [https://www.131002.net/siphash/siphash.pdf](https://www.131002.net/siphash/siphash.pdf)

## まとめ

vector、文字列、およびhash mapは、データを格納、アクセス、および変更する必要がある場合に、プログラムに必要な大量の機能を提供します。ここで解決する必要があるいくつかの演習があります。

* 整数のリストが与えられた場合、vectorを使用して平均値（平均値）、中央値（ソートされたとき、中央の値）、およびモード（最も頻繁に発生する値、hash mapはここで役立ちます ）のリスト
* 文字列をPig Latinに変換します。 各単語の最初の子音が単語の末尾に移動し、"ay"が追加されるので、"最初"は "irst-fay"になります。母音で始まる単語は末尾に"hay"が追加されます (“apple”は“apple-hay”になります)。UTF-8エンコーディングの詳細を覚えておいてください
* hash mapとvectorを使用して、ユーザーが社内の部門に従業員名を追加できるようにするテキストインタフェースを作成します。たとえば、Sallyをエンジニアリングに追加する」または「AmirをSalesに追加する」などとします。部門内の全員または部門内の全員のリストをアルファベット順に並べ替えることができます

標準ライブラリのAPIドキュメントでは、これらの演習に役立つベクトル、文字列、およびhash mapのメソッドについて説明しています。

操作が失敗するより複雑なプログラムになってきているので、エラー処理について議論するのに最適なタイミングです。
