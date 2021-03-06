# はじめに

Rustについての入門書、*The Rust Programming Language*へようこそ。Rustプログラミング言語は、より高速で信頼性の高いソフトウェアの作成に役立ちます。高水準の人間工学と低レベルの制御は、しばしばプログラミング言語の設計において偶然です。Rustはその競合に挑戦します。強力なバランスをとることで技術的な能力と優れた開発者経験を持つRustは、あなたにオプションを提供します。すべての手間をかけずに（例えば、メモリ使用量などの）低レベルの詳細を制御します。

## 誰が必要とするのか

Rustはさまざまな理由から多くの人々にとって理想的です。最も重要なグループのいくつかを見てみましょう。


### 開発チームにとって

Rustはさまざまなレベルのシステムプログラミング知識を持つ大規模な開発者チーム間のコラボレーションのための生産的なツールであることが証明されています。低レベルのコードでは、さまざまな微妙なバグが発生する可能性があります。これは、他の多くの言語では、経験豊かな開発者による広範なテストと慎重なコードレビューによってしか検出できません。Rustでは、コンパイラは次のことを拒否してゲートキーパーの役割を果たします。
並行性のバグを含むこれらの難解なバグをコードをコンパイルします。コンパイラと一緒に作業することで、チームはバグを追い出すのではなく、プログラムのロジックに集中して時間を費やすことができます。

Rustは現代の開発ツールをシステムプログラミングの世界にももたらします。

* 付属の依存マネージャーとビルドツールであるCargoは、Rustエコシステム全体で、無痛で一貫性のある依存関係の追加、コンパイル、および管理を行います。
* Rustfmtは開発者間で一貫したコーディングスタイルを保証します。
* TRust Language Serverはコード補完とインラインエラーメッセージの統合開発環境（IDE）の統合を実現します。

Rustエコシステムでこれらのツールや他のツールを使用することで、開発者はシステムレベルのコードを書く際に生産性を高めることができます。

### 学生にとって

Rustは学生やシステムコンセプトの学習に興味のある人のためのものです。Rustを使用して、オペレーティングシステム開発のようなトピックについて多くの人が学びました。コミュニティは非常に歓迎しており、学生の疑問に答えることができます。この本のような努力を通して、Rustチームは、より多くの人々、特にプログラミングに新しいシステムコンセプトにアクセスしやすくしたいと考えています。

### 会社にとって


大小さまざまな企業がさまざまな作業のために生産でRustを使用しています。これらのタスクには、コマンドラインツール、Webサービス、DevOpsツール、組み込みデバイス、オーディオとビデオの分析とトランスコード、暗号化通信、バイオインフォマティクス、検索エンジン、IoTアプリケーション、機械学習、Firefox Webブラウザの主要部分が含まれます。

### オープンソース開発者にとって

Rustは、Rustプログラミング言語、コミュニティ、開発者ツール、およびライブラリを構築したい人向けです。私たちはあなたにRust言語に貢献してもらいたいです。

### スピードと安定性を重視する人々にとって

Rustは言語のスピードと安定性を求めている人のためのものです。 スピードとは、Rustで作成できるプログラムのスピードとRustで書き込みできるスピードです。 Rustコンパイラのチェックは、機能の追加やリファクタリングによる安定性を保証します。 これは、これらのチェックがない言語の脆弱なレガシーコードとは対照的であり、開発者はしばしば変更することを恐れている。 手動で書かれたコードと同じくらい速く低レベルのコードにコンパイルする高レベルの機能をゼロコストの抽象化に努めて、Rustは安全なコードを高速なコードにするよう努めています。

Rust言語は他の多くのユーザーもサポートしたいと考えています。ここに挙げたものは、最大の利害関係者の一部にすぎません。 全体として、Rustの最大の野望は、安全性と生産性、スピードと人間工学を提供することにより、プログラマが何十年も受け入れてきたトレードオフを排除することです。その選択があなたのために働くかどうか試してみてください。

## この本の対象者

この本は、あなたが別のプログラミング言語でコードを書いたことを前提としていますが、どちらのプログラミング言語についても何も仮定していません。私たちは、幅広いプログラミングの背景にある人々が幅広くアクセスできるようにしました。私たちはプログラミングが何であるか、それについてどのように考えているかについて話すのに多くの時間を費やすことはありません。あなたがプログラミングに全く触れていないなら、プログラミングの入門書を特別に読んでいる本を読むことでより効果的です。

## この本の使い方

一般的に、この本はあなたが前から後に順にそれを読んでいることを前提としています。後の章では、以前の章の概念を基にしています。以前の章では、トピックの詳細を掘り下げていないかもしれません。我々は、通常、後の章でそのトピックを再訪します。

この本では、概念の章とプロジェクトの章の2種類の章があります。概念の章では、Rustの側面について学びます。 プロジェクトの章では、これまでに学んだことを応用して、小さなプログラムをまとめていきます。第2章、第12章、第20章はプロジェクトの章です。残りは概念の章です。

第1章では、Rustをインストールする方法、Hello Worldを書く方法について説明します。Cargo、Rustのパッケージマネージャーとビルドツールを使用する方法について説明します。第2章では、Rust言語の手引きを紹介します。ここでは概念を高レベルでカバーし、後の章では詳細を説明します。すぐに手を動かしたければ、第2章がその場所です。最初は、他のプログラミング言語と同様のRust機能を扱う第3章はスキップし、Rustの所有システムについては第4章を参照してください。しかし、あなたが次のものに進む前にすべての詳細を学ぶことを好む特に細心のある学習者ならば、第2章をスキップし、第3章に進み、第2章に戻り、プロジェクトは、あなたが学んだ詳細を適用します。

第5章では構造体とメソッドについて説明し、第6章ではenum、 `match`式、 `if let`制御フローの構造について説明します。 構造体と列挙型を使用して、Rustでカスタム型を作成します。

第7章では、Rustのモジュールシステムとコードとその公開API（Application Programming Interface）の整理に関するプライバシー規則について学びます。第8章ではベクトル、文字列、hash mapなど、標準ライブラリが提供する共通のコレクションデータ構造について説明します。第9章ではRustのエラー処理の哲学とテクニックについて解説します。

第10章では、generics、traits、およびライフタイムについて説明します。これにより、複数のタイプに適用されるコードを定義する権限が与えられます。第11章はすべてテストに関するもので、プログラムのロジックが正しいことを保証するためにRustの安全保証が必要な場合でもあります。第12章では、ファイル内のテキストを検索する`grep`コマンドラインツールから、独自の機能の実装を構築します。このために、前の章で説明した概念の多くを使用します。

第13章では関数型プログラミング言語に由来するRustの機能であるクロージャとイテレータについて説明します。第14章では、
ライブラリを他の人と共有するためのベストプラクティスについて深く説明します。
第15章では標準ライブラリが提供するスマートポインタとその機能を有効にする特徴について説明します。

第16章では並行プログラミングのさまざまなモデルを紹介し、Rustが複数のスレッドで勇敢にプログラムするのに役立つ方法について説明します。第17章ではRustイディオムとあなたが慣れ親しんでいるオブジェクト指向プログラミングの原理との比較を説明します。

第18章ではパターンとパターンのマッチングについて説明します。これはRustプログラム全体でアイデアを表現する強力な方法です。第19章では、安全でないRust、マクロ、ライフサイクル、トレイト、型、関数、クロージャなどの高度なトピックを紹介します。

第20章では低レベルのマルチスレッドWebサーバーを実装するプロジェクトを完成させます。

最後にいくつかの付録には言語についてのより有用な情報がリファレンスのような形式で含まれています。Appendix AはRustのキーワード、Appendix BはRustの演算子と記号、Appendix Cは標準ライブラリで提供される微分特性、付録Dは有用な開発ツール、Appendix EはRustエディションを説明しています。

この本を読むための全く間違った方法はありません：あなたはスキップしたい場合はそうしてください！
あなたが任意の混乱が発生した場合は、以前の章に戻ってジャンプする必要がある場合があります。 

<span id="ferris"></span>

Rustを学ぶプロセスの重要な部分は、コンパイラが表示するエラーメッセージを読む方法を学ぶことです。これらは作業コードに向かってあなたを導くでしょう。 したがって、コンパイラが各状況で表示されるエラーメッセージとともにコンパイルされないコードの例を多数提供します。 ランダムな例を入力して実行すると、コンパイルされないことがあります。 周囲のテキストを読んで、実行しようとしている例がエラーであるかどうかを確認してください。 Ferrisは、動作しないコードを区別するのにも役立ちます。

| Ferris                                                                 | Meaning                                          |
|------------------------------------------------------------------------|--------------------------------------------------|
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain">     | このコードはコンパイルされません！
                    |
| <img src="img/ferris/panics.svg" class="ferris-explain">               | このコードはpanics!                                |
| <img src="img/ferris/unsafe.svg" class="ferris-explain">               | このコードブロックには安全でないコードが含まれています
            |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain"> | このコードは、期待通りの動作をしません
 |

ほとんどの場合、コンパイルされないコードの正しいバージョンに導くでしょう。


## ソースコード

この本が生成されたソースファイルは[GitHub][book]にあります。

[book]: https://github.com/SIVIRA/rust-lang-book-ja/tree/translation/src-ja
