## コメント

すべてのプログラマーはコードを分かりやすくするために努力していますが時には説明も必要です。このような場合、プログラマはコンパイラが無視するソースコードにメモや*コメント*を残しますが、それはソースコードを読んでいる人は役に立つちます。

ここに簡単なコメントがあります。

```rust
// Hello, world.
```

Rustでは、コメントは2つのスラッシュで始まり行の終わりまで続きます。単一行を越えるコメントについては、次のように各行に`//`を入れる必要があります。

```rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```

コメントはコードを含む行の最後に置くこともできます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    let lucky_number = 7; // I’m feeling lucky today.
}
```

しかし、次のフォーマットで使用されることがよくあります。コメントを付けるコードの上の行にコメントが表示されます。

<span class="filename">ファイル名: src/main.rs</span>

```rust
fn main() {
    // I’m feeling lucky today.
    let lucky_number = 7;
}
```

Rustにはドキュメンテーションコメントという別の種類のコメントがあります。これについては第14章で説明します。