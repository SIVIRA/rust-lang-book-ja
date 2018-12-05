## `if let`による簡潔な制御フロー

`if let`構文は`if`と`let`を組み合わせて、あるパターンにマッチする値を処理し、残りのものは無視します。コードリスト6-6のプログラムが`Option<u8>`の値と一致するが、値が3の場合にのみコードを実行したいと考えているプログラムです。

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

<span class="caption">リスト 6-6: 値が `Some(3)`のときに実行するコードだけを扱う`match`</span>

`Some(3)`のマッチで何かをしたいですが、他の`Some<u8>`の値や`None`の値では何もしません。`match`式を満たすために、一つのバリアントだけを処理した後に`_ => ()`を追加する必要があります。これは、追加する定型コードです。

代わりに、`if let`を使ってこれをより短い方法で書くことができます。次のコードはコードリスト6-6の`match`と同じように動作します：

```rust
# let some_u8_value = Some(0u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

`if let`の構文はパターンと`=`で区切られた式を取ります。`match`と同じように動作します。式は`match`に与えられ、パターンは最初のarmです。

`if let`を使うと入力が少なく、インデントが少なくなり、定型化されたコードが少なくなりますが、`match`が強制する徹底的なチェックを失います。`match`と`let`のどちらかを選択するかどうかは、特定の状況で何をしているのか、簡潔さを得ることが網羅的なチェックを失うための適切なトレードオフであるかによって異なります。

言い換えると、値が1つのパターンと一致したときにコードを実行し、その後他のすべての値を無視する`match`の場合、`let`を糖衣構文と考えることができます。

`else`に`if let`を含めることができます。`else`と一緒になるコードブロックは、`if`と`else`と同等の`match`式で`_`ケースと一緒になるコードブロックと同じです。リスト6-4の`Coin`列挙型の定義を思い出してください。ここで`Quarter`バリアントも`UsState`値を保持しています。私たちが見ている25セント硬貨を数えたいと思ったら、25セント硬貨の状態をアナウンスすると、このような`match`式でそれを行うことができます。

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
# let coin = Coin::Penny;
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```

あるいは次のように`if let`と`else`式を使うことができます。

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
# let coin = Coin::Penny;
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

プログラムが`match`を使って表現するのが冗長すぎるロジックを持っている状況があるなら、`let`がRustのツールボックスにあることを忘れないでください。

## まとめ

ここでは、enumを使用して列挙値のセットの1つであるカスタムタイプを作成する方法について説明しました。標準ライブラリの `Option<T>`型がエラーを防ぐために型システムをどのように使うのかを示しました。列挙型の値にデータが含まれている場合、処理する必要があるケースの数に応じて、それらの値を抽出して使用するには`match`または`if let`を使用できます。

ここまででRustプログラムは構造体と列挙型を使用して、ドメイン内の概念を表現できるようになりました。APIで使用するカスタム型を作成すると、型の安全性が保証されます。コンパイラは各関数が期待する型の値のみを取得するようにします。

ユーザーに必要なものだけが公開されるように、使いやすいAPIをユーザーに提供するために、Rustのモジュールに目を向けるようにしましょう。