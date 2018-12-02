# 列挙型とパターンマッチング

この章では、*列挙型*とも呼ばれる*enumerations*を見ていきます。列挙型では可能な値を列挙して型を定義できます。まず、列挙型を定義して使用し列挙型がデータとともに意味をどのようにエンコードするかを示します。次に「Option」と呼ばれる特に有用な列挙型を探索します。この列挙型は、値が何かと何もないことを表現しています。次に、`match`式のパターンマッチングがどのようにenumの異なる値に対して異なるコードを実行するかを見ていきます。最後に`if let`構造体がコード内の列挙型を扱うために利用できる便利で簡潔な別のイディオムであることを説明します。

列挙型は多くの言語の機能ですが、機能は言語ごとに異なります。Rustの列挙型は、F#、OCaml、Haskellなどの関数型言語の*代数型*に最もよく似ています。