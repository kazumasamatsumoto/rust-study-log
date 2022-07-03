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
> これは、impl で型とトレイトをくっつける時にお互いが外部の場合で、何かしらのプロジェクトに 10 個ほどクレートを呼び出して、
> クレート１から１０までに同じ実装がある場合、コンパイラが「え？全部に同じ impl があるねんけど、どれ呼び出したらええん？」
> みたいな感じで困惑するので、それを防ぎましょうね？と言うことですかね？
> なのでそれを防ぐ方法として、型、トレイトのどちらか最低一つでも「独自」の定義した内容を盛り込みましょう！
> みたいな認識だと思います。

#### デフォルト実装

時として、全ての型の全メソッドに対して実装を要求するのではなく、トレイトの全てあるいは一部のメソッドに対してデフォルトの振る舞いがあると有用です。 そうすれば、特定の型にトレイトを実装する際、各メソッドのデフォルト実装を保持するかオーバーライドするか選べるわけです。

リスト 10-14 は、リスト 10-12 のように、メソッドシグニチャだけを定義するのではなく、 Summary トレイトの summarize メソッドにデフォルトの文字列を指定する方法を示しています。

ファイル名: src/lib.rs

```rust
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

リスト 10-14: summarize メソッドのデフォルト実装がある Summary トレイトの定義

独自の実装を定義するのではなく、デフォルト実装を利用して NewsArticle のインスタンスをまとめるには、 impl Summary for NewsArticle {}と空の impl ブロックを指定します。

もはや NewsArticle に直接 summarize メソッドを定義してはいませんが、私達はデフォルト実装を提供しており、 NewsArticle は Summary トレイトを実装すると指定しました。そのため、 NewsArticle のインスタンスに対して summarize メソッドを同じように呼び出すことができます。 このように:

```rust
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

summarize にデフォルト実装を用意しても、リスト 10-13 の Tweet の Summary 実装を変える必要はありません。 理由は、デフォルト実装をオーバーライドする記法はデフォルト実装のないトレイトメソッドを実装する記法と同じだからです。

デフォルト実装は、自らのトレイトのデフォルト実装を持たない他のメソッドを呼び出すことができます。 このようにすれば、トレイトは多くの有用な機能を提供しつつ、実装者は僅かな部分しか指定しなくて済むようになります。 例えば、Summary トレイトを、（実装者が）内容を実装しなければならない summarize_author メソッドを持つように定義し、 それから summarize_author メソッドを呼び出すデフォルト実装を持つ summarize メソッドを定義することもできます:

```rust
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

このバージョンの Summary を使用するために、型にトレイトを実装する際、実装する必要があるのは summarize_author だけです:

```rust
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

summarize_author 定義後、Tweet 構造体のインスタンスに対して summarize を呼び出せ、 summarize のデフォルト実装は、私達が提供した summarize_author の定義を呼び出すでしょう。 summarize_author を実装したので、追加のコードを書く必要なく、Summary トレイトは、 summarize メソッドの振る舞いを与えてくれました。

```rust
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

このコードは、1 new tweet: (Read more from @horse_ebooks...)（1 つの新しいツイート：（@horse_ebooks さんの文章をもっと読む））と出力します。

デフォルト実装を、そのメソッドをオーバーライドしている実装から呼び出すことはできないことに注意してください。

> 感想
>
> 鬼難しい。デフォルト実装のポイントがさっぱりわからん。
> とりあえず今のところわかっているのは、デフォルト実装という機能を使うと impl の時に宣言しなくてもいいことはわかった。
> ただ何がデフォルト実装で、何がデフォルト実装ではないのか？の区別がつかない。

トレイトは関数名と引数と返り値の型情報だけで構成される。

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

トレイトのデフォルト実装は関数名と引数と返り値の型情報と、関数の実行内容も記載されます。

```rust
fn summarize_author(&self) -> String; // トレイトの実装

pub trait Summary { // デフォルトトレイトの実装
    fn summarize(&self) -> String {
        // "（もっと読む）"
        String::from("(Read more...)")
    }
}
```

> デフォルト実装はトレイトの定義時に処理も記載しているかどうかで判別します。

### 引数としてのトレイト

トレイトを定義し実装する方法はわかったので、
トレイトを使っていろんな種類の型を受け付ける関数を定義する方法を学んでいきましょう。

たとえば、Listing 10-13 では、NewsArticle と Tweet 型に Summary トレイトを実装しました。
ここで、引数の item の summarize メソッドを呼ぶ関数 notify を定義することができます。
ただし、引数 item は Summary トレイトを実装しているような何らかの型であるとします。
このようなことをするためには、impl Trait 構文を使うことができます。

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

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
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

引数の item には、具体的な型の代わりに、impl キーワードとトレイト名を指定します。
この引数は、指定されたトレイトを実装しているあらゆる型を受け付けます。
notify の中身では、summarize のような、Summary トレイトに由来する item のあらゆるメソッドを呼び出すことができます。
私達は、notify を呼びだし、NewsArticle か Tweet のどんなインスタンスでも渡すことができます。
この関数を呼び出すときに、String や i32 のような他の型を渡すようなコードはコンパイルできません。
なぜなら、これらの型は Summary を実装していないからです。

> 引数に対してトレイトを実装することができて、引数にトレイトの振る舞いを付与することができるので、
> 実行内容にトレイトの振る舞いの関数を使用することができる。

#### トレイト境界構文

impl Trait 構文は単純なケースを解決しますが、実はより長いトレイト境界 (trait bound) と呼ばれる姿の糖衣構文 (syntax sugar) なのです。 それは以下のようなものです：

```rust
pub fn notify<T: Summary>(item: &T) {
    // 速報！ {}
    println!("Breaking news! {}", item.summarize());
}
```

この「より長い」姿は前節の例と等価ですが、より冗長です。
山カッコの中にジェネリックな型引数の宣言を書き、型引数の後ろにコロンを挟んでトレイト境界を置いています。

簡単なケースに対し、impl Trait 構文は便利で、コードを簡潔にしてくれます。
そうでないケースの場合、トレイト境界構文を使えば複雑な状態を表現できます。
たとえば、Summary を実装する 2 つのパラメータを持つような関数を考えることができます。
impl Trait 構文を使うとこのようになるでしょう：

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

この関数が受け取る item1 と item2 の型が（どちらも Summary を実装する限り）異なっても良いとするならば、
impl Trait は適切でしょう。
両方の引数が同じ型であることを強制することは、以下のようにトレイト境界を使ってのみ表現可能です：

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

引数である item1 と item2 の型としてジェネリックな型 T を指定しました。
これにより、item1 と item2 として関数に渡される値の具体的な型が同一でなければならない、
という制約を与えています。

> ここはあくまでもトレイトを元々の表現方法であって、簡潔に実装できるようになっているので、原理ぐらいにとどめておくほうがいい。

#### 複数のトレイト境界を+構文で指定する

複数のトレイト境界も指定できます。
たとえば、notify に summarize メソッドに加えて item の画面出力形式（ディスプレイフォーマット）を使わせたいとします。
その場合は、notify の定義に item は Display と Summary の両方を実装していなくてはならないと指定することになります。
これは、以下のように+構文で行うことができます：

```rust
pub fn notify(item: &(impl Summary + Display)) {
```

+構文はジェネリック型につけたトレイト境界に対しても使えます：

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

これら 2 つのトレイト境界が指定されていれば、notify の中では summarize を呼び出すことと、
{}を使って item をフォーマットすることの両方が行なえます。

#### where 句を使ったより明確なトレイト境界

あまりたくさんのトレイト境界を使うことには欠点もあります。
それぞれのジェネリック（な型）がそれぞれのトレイト境界をもつので、
複数のジェネリック型の引数をもつ関数は、関数名と引数リストの間に大量のトレイト境界に関する情報を含むことがあります。
これでは関数のシグネチャが読みにくくなってしまいます。
このため、Rust はトレイト境界を関数シグネチャの後の where 句の中で指定するという別の構文を用意しています。 なので、このように書く：

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

代わりに、where 句を使い、このように書くことができます：

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

この関数シグニチャは、よりさっぱりとしています。
トレイト境界を多く持たない関数と同じように、関数名、引数リスト、戻り値の型が一緒になって近くにあるからですね。

> 基本的に書きやすくするためにあるものと思っておけばいい。実際に使うときになったら考えましょう。

### トレイトを実装している型を返す

以下のように、impl Trait 構文を戻り値型のところで使うことにより、あるトレイトを実装する何らかの型を返すことができます。

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

戻り値の型として impl Summary を使うことにより、具体的な型が何かを言うことなく、
returns_summarizable 関数は Summary トレイトを実装している何らかの型を返すのだ、と指定することができます。
今回 returns_summarizable は Tweet を返しますが、この関数を呼び出すコードはそのことを知りません。

実装しているトレイトだけで戻り値型を指定できることは、13 章で学ぶ、クロージャとイテレータを扱うときに特に便利です。
クロージャとイテレータの作り出す型は、コンパイラだけが知っているものであったり、指定するには長すぎるものであったりします。
impl Trait 構文を使えば、非常に長い型を書くことなく、ある関数は Iterator トレイトを実装するある型を返すのだ、
と簡潔に指定することができます。

ただし、impl Trait は一種類の型を返す場合にのみ使えます。
たとえば、以下のように、戻り値の型は impl Summary で指定しつつ、NewsArticle か Tweet を返すようなコードは失敗します：

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
```

NewsArticle か Tweet を返すというのは、コンパイラの impl Trait 構文の実装まわりの制約により許されていません。
このような振る舞いをする関数を書く方法は、17 章のトレイトオブジェクトで異なる型の値を許容する節で学びます。

> 返り値のトレイトは一種類だけ。条件によって if 文制御ができない

### トレイト境界で largest 関数を修正する

ジェネリックな型引数の境界で使用したい振る舞いを指定する方法がわかったので、リスト 10-5 に戻って、
ジェネリックな型引数を使用する largest 関数の定義を修正しましょう！
最後にそのコードを実行しようとした時、 こんなエラーが出ていました:

```
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
  = note: `T` might need a bound for `std::cmp::PartialOrd`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10`.

To learn more, run the command again with --verbose.
```

largest の本体で、大なり演算子(>)を使用して型 T の 2 つの値を比較しようとしていました。
この演算子は、 標準ライブラリトレイトの std::cmp::PartialOrd でデフォルトメソッドとして定義されているので、
largest 関数が、比較できるあらゆる型のスライスに対して動くようにするためには、
T のトレイト境界に PartialOrd を指定する必要があります。
PartialOrd は prelude に含まれているので、これをスコープに導入する必要はありません。
largest のシグニチャを以下のように変えてください:

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

今回のコンパイルでは、別のエラーが出てきます：

```
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0508]: cannot move out of type `[T]`, a non-copy slice
（エラー[E0508]： 型`[T]`をもつ、非コピーのスライスからのムーブはできません）
 --> src/main.rs:2:23
  |
2 |     let mut largest = list[0];
  |                       ^^^^^^^
  |                       |
  |                       cannot move out of here
  |                       （ここからムーブすることはできません）
  |                       move occurs because `list[_]` has type `T`, which does not implement the `Copy` trait
  |                       （ムーブが発生するのは、`list[_]`は`T`という、`Copy`トレイトを実装しない型であるためです）
  |                       help: consider borrowing here: `&list[0]`
  |                       （助言：借用するようにしてみてはいかがですか： `&list[0]`）

error[E0507]: cannot move out of a shared reference
（エラー[E0507]：共有の参照からムーブはできません）
 --> src/main.rs:4:18
  |
4 |     for &item in list {
  |         -----    ^^^^
  |         ||
  |         |data moved here
  |         |（データがここでムーブされています）
  |         |move occurs because `item` has type `T`, which does not implement the `Copy` trait
  |         |（ムーブが発生するのは、`item`は`T`という、`Copy`トレイトを実装しない型であるためです）
  |         help: consider removing the `&`: `item`
  |         （助言：`&`を取り除いてみてはいかがですか： `item`）

error: aborting due to 2 previous errors

Some errors have detailed explanations: E0507, E0508.
For more information about an error, try `rustc --explain E0507`.
error: could not compile `chapter10`.

To learn more, run the command again with --verbose.
```

このエラーの鍵となる行は、cannot move out of type [T], a non-copy slice です。
ジェネリックでないバージョンの largest 関数では、最大の i32 か char を探そうとするだけでした。
第 4 章のスタックのみのデータ: コピー節で議論したように、i32 や char のようなサイズが既知の型はスタックに格納できるので、
Copy トレイトを実装しています。
しかし、largest 関数をジェネリックにすると、list 引数が Copy トレイトを実装しない型を含む可能性も出てきたのです。
結果として、list[0]から値を largest にムーブできず、このエラーに陥ったのです。

このコードを Copy トレイトを実装する型だけを使って呼び出すようにしたいなら、
T のトレイト境界に Copy を追加すればよいです！
リスト 10-15 は、関数に渡したスライスの値の型が、
i32 や char などのように PartialOrd と Copy を実装する限りコンパイルできる、
ジェネリックな largest 関数の完全なコードを示しています。

ファイル名: src/main.rs

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

リスト 10-15: PartialOrd と Copy トレイトを実装するあらゆるジェネリックな型に対して動く、 largest 関数の実際の定義

もし largest 関数を Copy を実装する型だけに制限したくなかったら、
T が Copy ではなく Clone というトレイト境界を持つと指定することもできます。
そうしたら、 largest 関数に所有権が欲しい時にスライスの各値をクローンできます。
clone 関数を使用するということは、
String のようなヒープデータを持つ型の場合により多くのヒープ確保が発生する可能性があることを意味します。
そして、大量のデータを取り扱っていたら、ヒープ確保には時間がかかることもあります。

largest の別の実装方法は、関数がスライスの T 値への参照を返すようにすることです。
戻り値の型を T ではなく&T に変え、それにより関数の本体を参照を返すように変更したら、
Clone や Copy トレイト境界は必要なくなり、ヒープ確保も避けられるでしょう。 これらの代替策をご自身で実装してみましょう！

### トレイト境界を使用して、メソッド実装を条件分けする

ジェネリックな型引数を持つ impl ブロックにトレイト境界を与えることで、 特定のトレイトを実装する型に対するメソッド実装を条件分けできます。例えば、 リスト 10-16 の型 Pair<T>は、常に new 関数を実装します。しかし、Pair<T>は、 内部の型 T が比較を可能にする PartialOrd トレイトと出力を可能にする Display トレイトを実装している時のみ、 cmp_display メソッドを実装します。

ファイル名: src/lib.rs

```rust
#![allow(unused)]
fn main() {
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
}
```

リスト 10-16: トレイト境界によってジェネリックな型に対するメソッド実装を条件分けする

また、別のトレイトを実装するあらゆる型に対するトレイト実装を条件分けすることもできます。 トレイト境界を満たすあらゆる型にトレイトを実装することは、ブランケット実装(blanket implementation)と呼ばれ、 Rust の標準ライブラリで広く使用されています。例を挙げれば、標準ライブラリは、 Display トレイトを実装するあらゆる型に ToString トレイトを実装しています。 標準ライブラリの impl ブロックは以下のような見た目です:

```
impl<T: Display> ToString for T {
    // --snip--
}
```

標準ライブラリにはこのブランケット実装があるので、Display トレイトを実装する任意の型に対して、 ToString トレイトで定義された to_string メソッドを呼び出せるのです。 例えば、整数は Display を実装するので、このように整数値を対応する String 値に変換できます:

```rust
#![allow(unused)]
fn main() {
let s = 3.to_string();
}
```

ブランケット実装は、トレイトのドキュメンテーションの「実装したもの」節に出現します。

トレイトとトレイト境界により、ジェネリックな型引数を使用して重複を減らしつつ、コンパイラに対して、 そのジェネリックな型に特定の振る舞いが欲しいことを指定するコードを書くことができます。 それからコンパイラは、トレイト境界の情報を活用してコードに使用された具体的な型が正しい振る舞いを提供しているか確認できます。 動的型付き言語では、その型に定義されていないメソッドを呼び出せば、実行時 (runtime) にエラーが出るでしょう。 しかし、Rust はこの種のエラーをコンパイル時に移したので、コードが動かせるようになる以前に問題を修正することを強制されるのです。 加えて、コンパイル時に既に確認したので、実行時の振る舞いを確認するコードを書かなくても済みます。 そうすることで、ジェネリクスの柔軟性を諦めることなくパフォーマンスを向上させます。

すでに使っている他のジェネリクスに、ライフタイムと呼ばれるものがあります。 ライフタイムは、型が欲しい振る舞いを保持していることではなく、必要な間だけ参照が有効であることを保証します。 ライフタイムがどうやってそれを行うかを見てみましょう。

> ひとまずトレイトのイメージが今は掴めました。
> 次はライフタイムです。