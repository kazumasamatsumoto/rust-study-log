# 2022 年 6 月 29 日

## 今日の学習内容

トレイト：共通の振る舞いを定義する

### トレイト:共通の振る舞いを定義する

トレイトは、Rust コンパイラに、特定の型に存在し、他の型と共有できる機能について知らせます。
トレイトを使用すると、共通の振る舞いを抽象的に定義できます。
トレイト境界を使用すると、あるジェネリックが、特定の振る舞いをもつあらゆる型になり得ることを指定できます。

> トレイトの機能としては
>
> ・Rust コンパイラに知らせる（特定の型に存在し、他の方と共有できる機能について）
>
> ・共通の振る舞いを抽象的に定義できます
>
> ・トレイト境界というのがある
>
> ・とあるジェネリックが、特定の振る舞いをもつあらゆる型になり得る

という機能があったりするということです。

> 注釈：違いはあるものの、トレイトは他の言語でよくインターフェイスと呼ばれる機能に類似しています。

#### トレイトを定義する

型の振る舞いは、その型に対して呼び出せるメソッドから構成されます。異なる型は、それらの型全てに対して同じメソッドを呼び出せるなら、同じ振る舞いを共有することになります。トレイト定義は、メソッドシグニチャをあるグループにまとめ、なんらかの目的を達成するのに必要な一連の振る舞いを定義する手段です。

> 基本的な考え方としては型の定義が違うけど、実際のロジックは同じという状況が発生すると、同じコードを再度書くのが DRY（Don't Repeat Yourself)の原則から外れるからやめた方がいいよということです。（別に使っても問題ないです。一緒に働く仲間からは嫌われる可能性があるので、仲良くしたい人はうまく書けないことを伝えておくか、共通化が必要になったときにどのようにしたらいいのか？テックリードあたりに確認するのがいいです）
>
> で、そのテックリードとかが「知るか！」という場合はあっていないので職場を離れましょう。基本的にコード戦略等は指揮をとる人に確認します。自分勝手なコードを書かないように、この時強い言葉（パンチラインの効いた言葉）を使う人とは仕事をするのは要注意です。（京都的な嫌味な言い回しも気をつけましょう）「結構学生の頃頑張ってたんですね。」とかは「お前のコードは学生レベルだからな」という言い回しですので、非常に気をつけましょう。

例えば、いろんな種類や量のテキストを保持する複数の構造体があるとしましょう:特定の場所から送られる新しいニュースを保持する`NewsArticle`と、新規ツイートか、リツイートか、はたまた他のツイートへのリプライなのかを示すメタデータを伴う最大で 280 文字までの`Tweet`です。

`NewsArticle`または`Tweet`インスタンスに保存されているデータのサマリーを表示できるメディアアグリゲータライブラリを作成します。これをするには、各型のサマリーが必要で、インスタンスで`summarize`メソッドを呼び出してサマリーを要求する必要があります。リスト 10-12 は、この振る舞いを表現する`Summary`トレイトの定義を表示しています。

> 要するにここではツイートやニュース記事の統計データと目次を表示できるライブラリを作ろうとします。アグリゲートは「集める」や「合計する」「集計する」という意味があります。なのでイメージとしては 2ch まとめサイトがいいかと思います。

ファイル名: src/lib.rs

```rust
# #![allow(unused)]
# fn main() {
pub trait Summary {
    fn summarize(&self) -> String;
}
# }
```

**リスト 10-12: `summarize`メソッドで提供される振る舞いからなる`Summary`トレイト**

ここでは、trait キーワード、それからトレイト名を使用してトレイトを定義していて、その名前は今回の場合、Summary です。波括弧の中にこのトレイトを実装する型の振る舞いを記述するメソッドシグニチャを定義し、今回の場合は、fn summarize(&self) -> String です。

> pub trait と宣言することで外部ファイルでも使用できる trait（振る舞い）を定義することができます。
> この場合の振る舞いは Summary と言う振る舞いでその中身は fn summarize(&self) -> String
> と言う返り値が文字列の配列になる形です。

メソッドシグニチャの後に、波括弧内に実装を提供する代わりに、セミコロンを使用しています。このトレイトを実装する型はそれぞれ、メソッドの本体に独自の振る舞いを提供しなければなりません。コンパイラにより、Summary トレイトを保持するあらゆる型に、このシグニチャと全く同じメソッド summarize が定義されていることが強制されます。

トレイトには、本体に複数のメソッドを含むことができます:メソッドシグニチャは行ごとに並べられ、各行はセミコロンで終わります。

#### トレイトを型に実装する

今や Summary トレイトを使用して目的の動作を定義できたので、
メディアアグリゲータでこれを型に実装できます。
リスト 10-13 は、Summary トレイトを NewsArticle 構造体上に実装したもので、
ヘッドライン、著者、そして地域情報を使って summarize の戻り値を作っています。
Tweet 構造体に関しては、ツイートの内容が既に 280 文字に制限されていると仮定して、
ユーザー名の後にツイートのテキスト全体が続くものとして summarize を定義します。

ファイル名: src/lib.rs

```rust
# #![allow(unused)]
# fn main() {
#    pub trait Summary {
#        fn summarize(&self) -> String
#    }
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
# }
```

型にトレイトを実装することは、普通のメソッドを実装することに似ています。
違いは、impl の後に、実装したいトレイトの名前を置き、それから for キーワード、
さらにトレイトの実装対象の型の名前を指定することです。
impl ブロック内に、トレイト定義で定義したメソッドシグニチャを置きます。
各シグニチャの後にセミコロンを追記するのではなく、波括弧を使用し、
メソッド本体に特定の型のトレイトのメソッドに欲しい特定の振る舞いを入れます。

トレイトを実装後、普通のメソッド同様に NewsArticle や Tweet のインスタンスに対してこのメソッドを呼び出せます。 こんな感じで:

```rust
# use chapter10::{self, Summary, Tweet};

# fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
#}
```

このコードは、1 new tweet: horse_ebooks: of course, as you probably already know, people と出力します。

```rust
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

> 感想メモ：
>
> ここで self.username と self.content の 2 つに関してフォーマットを実施すると言う内容になっています。
> ですので、Tweet については username と content については自身の文字列の引数を受け取って関数を実行します。
> ポイントはあくまでも共通化処理なので、一つずつの構造体のキー名に対して impl を実装することも可能ですが、
> こちらの方が都合がいいのでは？と言うのが感想です（甘いかもしれませんが。）

リスト 10-13 で Summary トレイトと NewArticle、Tweet 型を同じ lib.rs に定義したので、
全部同じスコープにあることに注目してください。
この lib.rs を aggregator と呼ばれるクレート専用にして、
誰か他の人が私たちのクレートの機能を活用して自分のライブラリのスコープに
定義された構造体に Summary トレイトを実装したいとしましょう。
まず、トレイトをスコープに取り込む必要があるでしょう。

use aggregator::Summary;

と指定してそれを行えば、これにより、自分の型に Summary を実装することが可能になるでしょう。
Summary トレイトは、他のクレートが実装するためには、公開トレイトである必要があり、
ここでは、リスト 10-12 の trait の前に、pub キーワードを置いたのでそうなっています。

トレイト実装で注意すべき制限の 1 つは、トレイトか対象の型が自分のクレートに固有(local)である時のみ、
型に対してトレイトを実装できるということです。
例えば、Display のような標準ライブラリのトレイトを aggregator クレートの機能の一部として、
Tweet のような独自の型に実装できます。
型 Tweet が aggregator クレートに固有だからです。
また、Summary を aggregator クレートで Vec<T>に対して実装することもできます。
トレイト Summary は、aggregator クレートに固有だからです。

しかし、外部のトレイトを外部の型に対して実装することはできません。
例として、aggregator クレート内で Vec<T>に対して Display トレイトを実装することはできません。
Display と Vec<T>は標準ライブラリで定義され、aggregator クレートに固有ではないからです。
この制限は、コヒーレンス(coherence)、
特に孤児のルール(orphan rule)と呼ばれるプログラムの特性の一部で、
親の型が存在しないためにそう命名されました。
この規則により、他の人のコードが自分のコードを壊したり、
その逆が起きないことを保証してくれます。
この規則がなければ、2 つのクレートが同じ型に対して同じトレイトを実装できてしまい、
コンパイラはどちらの実装を使うべきかわからなくなってしまうでしょう。

> ■ コヒーレンス(Coherence：可干渉性)
> コヒーレンスとは、波が重なり合ったときの干渉縞の作り易さを示します。
>
> 重なり合う波同士の位相、振幅に一定の関係がある場合は、合成された波も一定の位相、振幅をもつので干渉縞ができます。
> 干渉縞を作れる波をコヒーレントな波といい、レーザ光は代表的なコヒーレントな波（コヒーレント光）です。
> 一方、位相、振幅がランダムな波は干渉縞を作ることができず、インコヒーレントな波といいます。
> 電球、LED、SLD、ASE 光源などはインコヒーレントな波（インコヒーレント光）です。

> ちょっとここ難しいので図式化が必要です。
>
> ここの部分登場人物を整理しないといけないです。
>
> ・NewArticle と Tweet と言う二つの型を同じ lib.rs のファイルに定義している
>
> ・同じスコープ内にある
>
> ・この lib.rs を aggregator と呼ばれるクレート専用にして、誰かの他の人が私たちのクレート機能を活用して自分のライブラリのスコープに定義された構造体に Summary トレイトを実装したいとしましょう。
>
> ・まず、トレイトをスコープに取り込む必要があります。
>
> ・トレイトを他のクレートが実装するためには定義したトレイトを公開状態にする必要があります。
>
> ・トレイト実装で注意すべき制限の 1 つは、トレイトか対象の型が自分のクレートに固有（local）であるときのみ、型に対してトレイトを実装できると言うことです。
>
> ・Display のような標準ライブラリのトレイトを aggregator クレートの機能の一部として、Tweet のような独自の型に実装できます。
>
> ・型 Tweet が aggregator クレートに固有だからです。
>
> ・また Summary を aggregator クレートで Vec<T>に対して実装することもできます。
>
> ・トレイト Summary は、aggregator クレートに固有だからです。
>
> ・しかし、外部トレイトを外部の型に対して実装することはできません。例として、aggregator クレート内で Vec<T>に対して Display トレイトを実装することはできません。
>
> ・Display と Vec<T>は標準ライブラリで定義され、aggregator クレートに固有ではないからです。
>
> ・この制限は、コヒーレンス(coherence)、特に孤児のルール(orphan rule)と呼ばれるプログラムの特性の一部で、
>
> ・親の型が存在しないためにそう命名されました。この規則により、他の人のコードが自分のコードを壊したり、その逆が起きないことを保証してくれます。この規則がなければ、2 つのクレートが同じ型に対して同じトレイトを実装できてしまい、コンパイラはどちらの実装を使うべきかわからなくなってしましいます。

> ここのポイント
>
> これは、implで型とトレイトをくっつける時にお互いが外部の場合で、何かしらのプロジェクトに10個ほどクレートを呼び出して、
> クレート１から１０までに同じ実装がある場合、コンパイラが「え？全部に同じimplがあるねんけど、どれ呼び出したらええん？」
> みたいな感じで困惑するので、それを防ぎましょうね？と言うことですかね？
> なのでそれを防ぐ方法として、型、トレイトのどちらか最低一つでも「独自」の定義した内容を盛り込みましょう！
> みたいな認識だと思います。

#### デフォルト実装
時として、全ての型の全メソッドに対して実装を要求するのではなく、トレイトの全てあるいは一部のメソッドに対してデフォルトの振る舞いがあると有用です。 そうすれば、特定の型にトレイトを実装する際、各メソッドのデフォルト実装を保持するかオーバーライドするか選べるわけです。

リスト10-14は、リスト10-12のように、メソッドシグニチャだけを定義するのではなく、 Summaryトレイトのsummarizeメソッドにデフォルトの文字列を指定する方法を示しています。

ファイル名: src/lib.rs

``` rust
# #![allow(unused)]
# fn main() {
pub trait Summary {
    fn summarize(&self) -> String {
        // "（もっと読む）"
        String::from("(Read more...)")
    }
}
#
# pub struct NewsArticle {
#    pub headline: String,
#    pub location: String,
#    pub author: String,
#    pub content: String,
#}
#
# impl Summary for NewsArticle {}
#
#pub struct Tweet {
#    pub username: String,
#    pub content: String,
#    pub reply: bool,
#    pub retweet: bool,
#}
#
#impl Summary for Tweet {
#    fn summarize(&self) -> String {
#        format!("{}: {}", self.username, self.content)
#    }
#}
#}
```

リスト10-14: summarizeメソッドのデフォルト実装があるSummaryトレイトの定義

独自の実装を定義するのではなく、デフォルト実装を利用してNewsArticleのインスタンスをまとめるには、 impl Summary for NewsArticle {}と空のimplブロックを指定します。

もはやNewsArticleに直接summarizeメソッドを定義してはいませんが、私達はデフォルト実装を提供しており、 NewsArticleはSummaryトレイトを実装すると指定しました。そのため、 NewsArticleのインスタンスに対してsummarizeメソッドを同じように呼び出すことができます。 このように:

``` rust
#use chapter10::{self, NewsArticle, Summary};
#
#fn main() {
    let article = NewsArticle {
        // ペンギンチームがスタンレーカップチャンピオンシップを勝ち取る！
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        // アメリカ、ペンシルベニア州、ピッツバーグ
        location: String::from("Pittsburgh, PA, USA"),
        // アイスバーグ
        author: String::from("Iceburgh"),
        // ピッツバーグ・ペンギンが再度NHL(National Hockey League)で最強のホッケーチームになった
        content: String::from(
            "The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
#}
```

このコードは、New article available! (Read more...)（新しい記事があります！（もっと読む））と出力します。

summarizeにデフォルト実装を用意しても、リスト10-13のTweetのSummary実装を変える必要はありません。 理由は、デフォルト実装をオーバーライドする記法はデフォルト実装のないトレイトメソッドを実装する記法と同じだからです。

デフォルト実装は、自らのトレイトのデフォルト実装を持たない他のメソッドを呼び出すことができます。 このようにすれば、トレイトは多くの有用な機能を提供しつつ、実装者は僅かな部分しか指定しなくて済むようになります。 例えば、Summaryトレイトを、（実装者が）内容を実装しなければならないsummarize_authorメソッドを持つように定義し、 それからsummarize_authorメソッドを呼び出すデフォルト実装を持つsummarizeメソッドを定義することもできます:


``` rust
##![allow(unused)]
#fn main() {
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        // "（{}さんの文章をもっと読む）"
        format!("(Read more from {}...)", self.summarize_author())
    }
}
#
#pub struct Tweet {
#    pub username: String,
#    pub content: String,
#    pub reply: bool,
#    pub retweet: bool,
#}
#
#impl Summary for Tweet {
#    fn summarize_author(&self) -> String {
#        format!("@{}", self.username)
#    }
#}
#}
```

このバージョンのSummaryを使用するために、型にトレイトを実装する際、実装する必要があるのはsummarize_authorだけです:

``` rust
#pub trait Summary {
#    fn summarize_author(&self) -> String;
#
#    fn summarize(&self) -> String {
#        // "（{}さんの文章をもっと読む）"
#        format!("(Read more from {}...)", self.summarize_author())
#    }
#}
#
#pub struct Tweet {
#    pub username: String,
#    pub content: String,
#    pub reply: bool,
#    pub retweet: bool,
#}
#
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

summarize_author定義後、Tweet構造体のインスタンスに対してsummarizeを呼び出せ、 summarizeのデフォルト実装は、私達が提供したsummarize_authorの定義を呼び出すでしょう。 summarize_authorを実装したので、追加のコードを書く必要なく、Summaryトレイトは、 summarizeメソッドの振る舞いを与えてくれました。

``` rust
#use chapter10::{self, Summary, Tweet};
#
#fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
#}
```

このコードは、1 new tweet: (Read more from @horse_ebooks...)（1つの新しいツイート：（@horse_ebooksさんの文章をもっと読む））と出力します。

デフォルト実装を、そのメソッドをオーバーライドしている実装から呼び出すことはできないことに注意してください。

> 感想
>
> 鬼難しい。デフォルト実装のポイントがさっぱりわからん。
> とりあえず今のところわかっているのは、デフォルト実装という機能を使うとimplの時に宣言しなくてもいいことはわかった。
> ただ何がデフォルト実装で、何がデフォルト実装ではないのか？の区別がつかない。