# リファクタリングしてモジュール性とエラー処理を向上させる
プログラムを改善するために、プログラムの構造と起こりうるエラーに対処する方法に関連する4つの問題を修正していきましょう。

1番目は、main関数が2つの仕事を受け持っていることです: 引数を解析し、ファイルを開いています。 このような小さな関数なら、これは、大した問題ではありませんが、main内でプログラムを巨大化させ続けたら、 main関数が扱う個別の仕事の数も増えていきます。関数が責任を受け持つごとに、 正しいことを確認しにくくなり、テストも行いづらくなり、機能を壊さずに変更するのも困難になっていきます。 機能を小分けして、各関数が1つの仕事のみに責任を持つようにするのが最善です。

この問題は、2番目の問題にも結びついています: queryとfilenameはプログラムの設定用変数ですが、 fやcontentsといった変数は、プログラムのロジックを担っています。mainが長くなるほど、 スコープに入れるべき変数も増えます。そして、スコープにある変数が増えれば、各々の目的を追うのも大変になるわけです。 設定用変数を一つの構造に押し込め、目的を明瞭化するのが最善です。

3番目の問題は、ファイルを開き損ねた時にexpectを使ってエラーメッセージを出力しているのに、 エラーメッセージがファイルが見つかりませんでしたとしか表示しないことです。 ファイルを開く行為は、ファイルが存在しない以外にもいろんな方法で失敗することがあります: 例えば、ファイルは存在するかもしれないけれど、開く権限がないかもしれないなどです。 現時点では、そのような状況になった時、「ファイルが見つかりませんでした」というエラーメッセージを出力し、 これはユーザに間違った情報を与えるのです。

4番目は、異なるエラーを処理するのにexpectを繰り返し使用しているので、ユーザが十分な数の引数を渡さずにプログラムを起動した時に、 問題を明確に説明しない「範囲外アクセス(index out of bounds)」というエラーがRustから得られることです。 エラー処理のコードが全て1箇所に存在し、将来エラー処理ロジックが変更になった時に、 メンテナンス者が1箇所のコードのみを考慮すればいいようにするのが最善でしょう。 エラー処理コードが1箇所にあれば、エンドユーザにとって意味のあるメッセージを出力していることを確認することにもつながります。

プロジェクトをリファクタリングして、これら4つの問題を扱いましょう。

## バイナリプロジェクトの責任の分離
main関数に複数の仕事の責任を割り当てるという構造上の問題は、多くのバイナリプロジェクトでありふれています。 結果として、mainが肥大化し始めた際にバイナリプログラムの個別の責任を分割するためにガイドラインとして活用できる工程をRustコミュニティは、 開発しました。この工程は、以下のような手順になっています:

- プログラムをmain.rsとlib.rsに分け、ロジックをlib.rsに移動する。
- コマンドライン引数の解析ロジックが小規模な限り、main.rsに置いても良い。
- コマンドライン引数の解析ロジックが複雑化の様相を呈し始めたら、main.rsから抽出してlib.rsに移動する。

この工程の後にmain関数に残る責任は以下に限定される:

- 引数の値でコマンドライン引数の解析ロジックを呼び出す
- 他のあらゆる設定を行う
- lib.rsのrun関数を呼び出す
- runがエラーを返した時に処理する

このパターンは、責任の分離についてです: main.rsはプログラムの実行を行い、 そして、lib.rsが手にある仕事のロジック全てを扱います。main関数を直接テストすることはできないので、 この構造により、プログラムのロジック全てをlib.rsの関数に移すことでテストできるようになります。 main.rsに残る唯一のコードは、読めばその正当性が評価できるだけ小規模になるでしょう。 この工程に従って、プログラムのやり直しをしましょう。

## 引数解析器を抽出する
引数解析の機能をmainが呼び出す関数に抽出して、コマンドライン引数解析ロジックをsrc/lib.rsに移動する準備をします。 リスト12-5に新しい関数parse_configを呼び出すmainの冒頭部を示し、 この新しい関数は今だけsrc/main.rsに定義します。

ファイル名: src/main.rs

``` rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, filename) = parse_config(&args);

    // --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let filename = &args[2];

    (query, filename)
}
```

リスト12-5: mainからparse_config関数を抽出する

それでもまだ、コマンドライン引数をベクタに集結させていますが、main関数内で引数の値の添え字1を変数queryに、 添え字2を変数filenameに代入する代わりに、ベクタ全体をparse_config関数に渡しています。 そして、parse_config関数にはどの引数がどの変数に入り、それらの値をmainに返すというロジックが存在します。 まだmain内にqueryとfilenameという変数を生成していますが、もうmainは、 コマンドライン引数と変数がどう対応するかを決定する責任は持ちません。

このやり直しは、私たちの小規模なプログラムにはやりすぎに思えるかもしれませんが、 少しずつ段階的にリファクタリングしているのです。この変更後、プログラムを再度実行して、 引数解析がまだ動作していることを実証してください。問題が発生した時に原因を特定する助けにするために頻繁に進捗を確認するのはいいことです。

## 設定値をまとめる
もう少しparse_config関数を改善することができます。現時点では、タプルを返していますが、 即座にタプルを分解して再度個別の値にしています。これは、正しい抽象化をまだできていないかもしれない兆候です。

まだ改善の余地があると示してくれる他の徴候は、parse_configのconfigの部分であり、 返却している二つの値は関係があり、一つの設定値の一部にどちらもなることを暗示しています。 現状では、一つのタプルにまとめていること以外、この意味をデータの構造に載せていません; この二つの値を1構造体に置き換え、構造体のフィールドそれぞれに意味のある名前をつけることもできるでしょう。 そうすることで将来このコードのメンテナンス者が、異なる値が相互に関係する仕方や、目的を理解しやすくできるでしょう。

> 注釈: この複雑型(complex type)がより適切な時に組み込みの値を使うアンチパターンを、 primitive obsession(訳注: 初めて聞いた表現。組み込み型強迫観念といったところだろうか)と呼ぶ人もいます。

> 要するに構造体定義をします。

``` rust
use std::env;
use std::fs::File;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    let mut f = File::open(config.filename).expect("file not found");

    // --snip--
}

struct Config {
    query: String,
    filename: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let filename = args[2].clone();

    Config { query, filename }
}
```

リスト12-6: parse_configをリファクタリングしてConfig構造体のインスタンスを返す

queryとfilenameというフィールドを持つよう定義されたConfigという構造体を追加しました。 parse_configのシグニチャは、これでConfig値を返すと示すようになりました。parse_configの本体では、 以前はargsのString値を参照する文字列スライスを返していましたが、 今では所有するString値を含むようにConfigを定義しています。mainのargs変数は引数値の所有者であり、 parse_config関数だけに借用させていますが、これはConfigがargsの値の所有権を奪おうとしたら、 Rustの借用規則に違反してしまうことを意味します。

Stringのデータは、多くの異なる手法で管理できますが、最も単純だけれどもどこか非効率的な手段は、 値に対してcloneメソッドを呼び出すことです。これにより、Configインスタンスが所有するデータの総コピーが生成されるので、 文字列データへの参照を保持するよりも時間とメモリを消費します。ですが、データをクローンすることで、 コードがとても素直にもなります。というのも、参照のライフタイムを管理する必要がないからです。 つまり、この場面において、少々のパフォーマンスを犠牲にして単純性を得るのは、価値のある代償です。

> cloneを使用する代償
>
> 実行時コストのためにcloneを使用して所有権問題を解消するのを避ける傾向が多くのRustaceanにあります。 第13章で、この種の状況においてより効率的なメソッドの使用法を学ぶでしょう。ですがとりあえずは、 これらのコピーをするのは1回だけですし、ファイル名とクエリ文字列は非常に小さなものなので、 いくつかの文字列をコピーして進捗するのは良しとしましょう。最初の通り道でコードを究極的に効率化しようとするよりも、 ちょっと非効率的でも動くプログラムを用意する方がいいでしょう。もっとRustの経験を積めば、 最も効率的な解決法から開始することも簡単になるでしょうが、今は、cloneを呼び出すことは完璧に受け入れられることです。

mainを更新したので、parse_configから返されたConfigのインスタンスをconfigという変数に置くようになり、 以前は個別のqueryとfilename変数を使用していたコードを更新したので、代わりにConfig構造体のフィールドを使用するようになりました。

これでコードはqueryとfilenameが関連していることと、その目的がプログラムの振る舞い方を設定するということをより明確に伝えます。 これらの値を使用するあらゆるコードは、configインスタンスの目的の名前を冠したフィールドにそれらを発見することを把握しています。

## Configのコンストラクタを作成する
ここまでで、コマンドライン引数を解析する責任を負ったロジックをmainから抽出し、parse_config関数に配置しました。 そうすることでqueryとfilenameの値が関連し、その関係性がコードに載っていることを確認する助けになりました。 それからConfig構造体を追加してqueryとfilenameの関係する目的を名前付けし、 構造体のフィールド名としてparse_config関数からその値の名前を返すことができています。

したがって、今やparse_config関数の目的はConfigインスタンスを生成することになったので、 parse_configをただの関数からConfig構造体に紐づくnewという関数に変えることができます。 この変更を行うことで、コードがより慣用的になります。Stringなどの標準ライブラリの型のインスタンスを、 String::newを呼び出すことで生成できます。同様に、parse_configをConfigに紐づくnew関数に変えれば、 Config::newを呼び出すことでConfigのインスタンスを生成できるようになります。リスト12-7が、 行う必要のある変更を示しています。

ファイル名: src/main.rs

> structに対してimplを実装します。

``` rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    // --snip--
}

struct Config {
    query: String,
    filename: String,
}

// --snip--

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let filename = args[2].clone();

        Config { query, filename }
    }
}
```

リスト12-7: parse_configをConfig::newに変える

parse_configを呼び出していたmainを代わりにConfig::newを呼び出すように更新しました。 parse_configの名前をnewに変え、implブロックに入れ込んだので、new関数とConfigが紐づくようになりました。 再度このコードをコンパイルしてみて、動作することを確かめてください。

## エラー処理を修正する
さて、エラー処理の修正に取り掛かりましょう。ベクタが2個以下の要素しか含んでいないときにargsベクタの添え字1か2にアクセスしようとすると、 プログラムがパニックすることを思い出してください。試しに引数なしでプログラムを実行してください。すると、こんな感じになります:

```
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'index out of bounds: the len is 1
but the index is 1', src/main.rs:29:21
(スレッド'main'は、「境界外アクセス: 長さは1なのに添え字も1です」でパニックしました)
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

境界外アクセス: 長さは1なのに添え字も1ですという行は、プログラマ向けのエラーメッセージです。 エンドユーザが起きたことと代わりにすべきことを理解する手助けにはならないでしょう。これを今修正しましょう。


## エラーメッセージを改善する
リスト12-8で、new関数に、添え字1と2にアクセスする前にスライスが十分長いことを実証するチェックを追加しています。 スライスの長さが十分でなければ、プログラムはパニックし、境界外インデックスよりもいいエラーメッセージを表示します。

ファイル名: src/main.rs

```
// --snip--
fn new(args: &[String]) -> Config {
    if args.len() < 3 {
        // 引数の数が足りません
        panic!("not enough arguments");
    }
    // --snip--
```

リスト12-8: 引数の数のチェックを追加する

このコードは、リスト9-9で記述したvalue引数が正常な値の範囲外だった時にpanic!を呼び出したGuess::new関数と似ています。 ここでは、値の範囲を確かめる代わりに、argsの長さが少なくとも3であることを確かめていて、 関数の残りの部分は、この条件が満たされているという前提のもとで処理を行うことができます。 argsに2要素以下しかなければ、この条件は真になり、panic!マクロを呼び出して、即座にプログラムを終了させます。

では、newのこの追加の数行がある状態で、再度引数なしでプログラムを走らせ、エラーがどんな見た目か確かめましょう:

```
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'not enough arguments', src/main.rs:30:12
(スレッド'main'は「引数が足りません」でパニックしました)
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

> これでエラー文が変わりました

この出力の方がマシです: これでエラーメッセージが合理的になりました。ですが、 ユーザに与えたくない追加の情報も含まれてしまっています。おそらく、 ここではリスト9-9で使用したテクニックを使用するのは最善ではありません: panic!の呼び出しは、第9章で議論したように、使用の問題よりもプログラミング上の問題により適しています。 代わりに、第9章で学んだもう一つのテクニックを使用することができます。成功か失敗かを示唆するResultを返すことです。

## panic!を呼び出す代わりにnewからResultを返す
代わりに、成功時にはConfigインスタンスを含み、エラー時には問題に言及するResult値を返すことができます。 Config::newがmainと対話する時、Result型を使用して問題があったと信号を送ることができます。 それからmainを変更して、panic!呼び出しが引き起こしていたthread 'main'とRUST_BACKTRACEに関する周囲のテキストがない、 ユーザ向けのより実用的なエラーにErr列挙子を変換することができます。

リスト12-9は、Config::newの戻り値に必要な変更とResultを返すのに必要な関数の本体を示しています。 mainも更新するまで、これはコンパイルできないことに注意してください。その更新は次のリストで行います。

ファイル名: src/main.rs

```rust
impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config { query, filename })
    }
}
```

> panicをreturn Errにしました。
>
> これでユーザーに対してエラーを表示することができます。

リスト12-9: Config::newからResultを返却する

new関数は、これで、成功時にはConfigインスタンスを、エラー時には&'static strを伴うResultを返すようになりました。 第10章の「静的ライフタイム」節から&'static strは文字列リテラルの型であることを思い出してください。 これは、今はエラーメッセージの型になっています。

new関数の本体で2つ変更を行いました: 十分な数の引数をユーザが渡さなかった場合にpanic!を呼び出す代わりに、 今はErr値を返し、Config戻り値をOkに包んでいます。これらの変更により、関数が新しい型シグニチャに適合するわけです。

Config::newからErr値を返すことにより、main関数は、new関数から返ってくるResult値を処理し、 エラー時により綺麗にプロセスから抜け出すことができます。

## Config::newを呼び出し、エラーを処理する
エラーケースを処理し、ユーザフレンドリーなメッセージを出力するために、mainを更新して、 リスト12-10に示したようにConfig::newから返されているResultを処理する必要があります。 また、panic!からコマンドラインツールを0以外のエラーコードで抜け出す責任も奪い取り、 手作業でそれも実装します。0以外の終了コードは、 我々のプログラムを呼び出したプロセスにプログラムがエラー状態で終了したことを通知する慣習です。

ファイル名: src/main.rs

```rust
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        // 引数解析時に問題
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
```

> この実行失敗時の処理は結構覚えておいてもいいかと。

リスト12-10: 新しいConfig作成に失敗したら、エラーコードで終了する

このリストにおいて、以前には講義していないメソッドを使用しました: unwrap_or_elseです。 これは標準ライブラリでResult<T, E>に定義されています。unwrap_or_elseを使うことで、 panic!ではない何らか独自のエラー処理を定義できるのです。このResultがOk値だったら、 このメソッドの振る舞いはunwrapに似ています: Okが包んでいる中身の値を返すのです。 しかし、値がErr値なら、このメソッドは、クロージャ内でコードを呼び出し、 クロージャは私たちが定義し、引数としてunwrap_or_elseに渡す匿名関数です。クロージャについては第13章で詳しく講義します。 とりあえず、unwrap_or_elseは、今回リスト12-9で追加したnot enough argumentsという静的文字列のErrの中身を、 縦棒の間に出現するerr引数のクロージャに渡していることだけ知っておく必要があります。 クロージャのコードはそれから、実行された時にerr値を使用できます。

新規use行を追加して標準ライブラリからprocessをインポートしました。クロージャ内のエラー時に走るコードは、 たった2行です: errの値を出力し、それからprocess::exitを呼び出します。process::exit関数は、 即座にプログラムを停止させ、渡された数字を終了コードとして返します。これは、リスト12-8で使用したpanic!ベースの処理と似ていますが、 もう余計な出力はされません。試しましょう:

```
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48 secs
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

素晴らしい！この出力の方が遥かにユーザに優しいです。

## mainからロジックを抽出する
これで設定解析のリファクタリングが終了したので、プログラムのロジックに目を向けましょう。 「バイナリプロジェクトの責任の分離」で述べたように、 現在main関数に存在する設定のセットアップやエラー処理に関わらない全てのロジックを保持することになるrunという関数を抽出します。 やり終わったら、mainは簡潔かつ視察で確かめやすくなり、他のロジック全部に対してテストを書くことができるでしょう。

リスト12-11は、抜き出したrun関数を示しています。今は少しずつ段階的に関数を抽出する改善を行っています。 それでも、src/main.rsに関数を定義していきます。

ファイル名: src/main.rs

``` rust
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    run(config);
}

fn run(config: Config) {
    let mut f = File::open(config.filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}

// --snip--
```

リスト12-11: 残りのプログラムロジックを含むrun関数を抽出する

これでrun関数は、ファイル読み込みから始まるmain関数の残りのロジック全てを含むようになりました。 このrun関数は、引数にConfigインスタンスを取ります。

## run関数からエラーを返す
残りのプログラムロジックがrun関数に隔離されたので、リスト12-9のConfig::newのように、 エラー処理を改善することができます。expectを呼び出してプログラムにパニックさせる代わりに、 run関数は、何か問題が起きた時にResult<T, E>を返します。これにより、 さらにエラー処理周りのロジックをユーザに優しい形でmainに統合することができます。 リスト12-12にシグニチャとrun本体に必要な変更を示しています。

ファイル名: src/main.rs

```rust
use std::error::Error;

// --snip--

fn run(config: Config) -> Result<(), Box<Error>> {
    let contents = fs::read_to_string(config.filename)?;
    println!("With text:\n{}", contents);

    Ok(())
}
```

リスト12-12: run関数を変更してResultを返す

ここでは、3つの大きな変更を行いました。まず、run関数の戻り値をResult<(), Box<Error>>に変えました。 この関数は、以前はユニット型、()を返していて、それをOkの場合に返される値として残しました。

エラー型については、トレイトオブジェクトのBox<Error>を使用しました(同時に冒頭でuse文により、 std::error::Errorをスコープに導入しています)。トレイトオブジェクトについては、第17章で講義します。 とりあえず、Box<Error>は、関数がErrorトレイトを実装する型を返すことを意味しますが、 戻り値の型を具体的に指定しなくても良いことを知っておいてください。これにより、 エラーケースによって異なる型のエラー値を返す柔軟性を得ます。

2番目に、expectの呼び出しよりも?演算子を選択して取り除きました。第9章で語りましたね。 エラーでパニックするのではなく、?演算子は呼び出し元が処理できるように、現在の関数からエラー値を返します。

3番目に、run関数は今、成功時にOk値を返すようになりました。run関数の成功型は、 シグニチャで()と定義したので、ユニット型の値をOk値に包む必要があります。 最初は、このOk(())という記法は奇妙に見えるかもしれませんが、このように()を使うことは、 runを副作用のためだけに呼び出していると示唆する慣習的な方法です; 必要な値は返しません。

このコードを実行すると、コンパイルは通るものの、警告が表示されるでしょう:

```
warning: unused `std::result::Result` which must be used
(警告: 使用されなければならない`std::result::Result`が未使用です)
  --> src/main.rs:18:5
   |
18 |     run(config);
   |     ^^^^^^^^^^^^
= note: #[warn(unused_must_use)] on by default
```

コンパイラは、コードがResult値を無視していると教えてくれて、このResult値は、 エラーが発生したと示唆しているかもしれません。しかし、エラーがあったか確認するつもりはありませんが、 コンパイラは、ここにエラー処理コードを書くつもりだったんじゃないかと思い出させてくれています！ 今、その問題を改修しましょう。

## mainでrunから返ってきたエラーを処理する
リスト12-10のConfig::newに対して行った方法に似たテクニックを使用してエラーを確認し、扱いますが、 少し違いがあります:

ファイル名: src/main.rs

``` rust
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    if let Err(e) = run(config) {
        println!("Application error: {}", e);

        process::exit(1);
    }
}
```

unwrap_or_elseではなく、if letでrunがErr値を返したかどうかを確認し、そうならprocess::exit(1)を呼び出しています。 run関数は、Config::newがConfigインスタンスを返すのと同じようにunwrapしたい値を返すことはありません。 runは成功時に()を返すので、エラーを検知することにのみ興味があり、()でしかないので、 unwrap_or_elseに包まれた値を返してもらう必要はないのです。

if letとunwrap_or_else関数の中身はどちらも同じです: エラーを出力して終了します。

## コードをライブラリクレートに分割する
ここまでminigrepは良さそうですね！では、テストを行え、src/main.rsファイルの責任が減らせるように、 src/main.rsファイルを分割し、一部のコードをsrc/lib.rsファイルに置きましょう。

main関数以外のコード全部をsrc/main.rsからsrc/lib.rsに移動しましょう:

- run関数定義
- 関係するuse文
- Configの定義
- Config::new関数定義

src/lib.rsの中身にはリスト12-13に示したようなシグニチャがあるはずです(関数の本体は簡潔性のために省略しました)。 リスト12-14でsrc/main.rsに変更を加えるまで、このコードはコンパイルできないことに注意してください。

ファイル名: src/lib.rs

``` rust
use std::error::Error;
use std::fs::File;
use std::io::prelude::*;

pub struct Config {
    pub query: String,
    pub filename: String,
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        // --snip--
    }
}

pub fn run(config: Config) -> Result<(), Box<Error>> {
    // --snip--
}
```

リスト12-13: Configとrunをsrc/lib.rsに移動する

ここでは、寛大にpubを使用しています: Configのフィールドとnewメソッドとrun関数です。 これでテスト可能な公開APIのあるライブラリクレートができました！

さて、src/lib.rsに移動したコードをsrc/main.rsのバイナリクレートのスコープに持っていく必要があります。 リスト12-14に示したようにですね。

ファイル名: src/main.rs

```rust
use std::env;
use std::process;

mod lib;
use lib::{Config, run};


fn main() {
    // --snip--
    if let Err(e) = minigrep::run(config) {
        // --snip--
    }
}
```

リスト12-14: minigrepクレートをsrc/main.rsのスコープに持っていく

ライブラリクレートをバイナリクレートに持っていくのに、extern crate minigrepを使用しています。 それからuse minigrep::Config行を追加してConfig型をスコープに持ってきて、 run関数にクレート名を接頭辞として付けます。これで全機能が連結され、動くはずです。 cargo runでプログラムを走らせて、すべてがうまくいっていることを確かめてください。

ふう！作業量が多かったですね。ですが、将来成功する準備はできています。 もう、エラー処理は遥かに楽になり、コードのモジュール化もできました。 ここから先の作業は、ほぼsrc/lib.rsで完結するでしょう。

古いコードでは大変だけれども、新しいコードでは楽なことをして新発見のモジュール性を活用しましょう: テストを書くのです！

> extern crateという表現はせずに他のファイルの場合はmod宣言で使用できます。