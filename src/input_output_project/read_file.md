# ファイルを読み込む
では、filenameコマンドライン引数で指定されたファイルを読み込む機能を追加しましょう。 まず、テスト実行するためのサンプルファイルが必要ですね: minigrepが動作していることを確かめるために使用するのに最適なファイルは、 複数行にわたって同じ単語の繰り返しのある少量のテキストです。リスト12-3は、 うまくいくであろうエミリー・ディキンソン(Emily Dickinson)の詩です！ プロジェクトのルート階層にpoem.txtというファイルを作成し、この詩「私は誰でもない！あなたは誰？」を入力してください。

ファイル名: poem.txt

```
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

> このポエムをpoem.txtに貼り付けて保存してください。

リスト12-3: エミリー・ディキンソンの詩は、いいテストケースになる

テキストを適当な場所に置いて、src/main.rsを編集し、ファイルを開くコードを追加してください。 リスト12-4に示したようにですね。

ファイル名: src/main.rs

``` rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", filename);

    let contents = fs::read_to_string(filename).expect("something went wrong reading the file");
    // テキストは\n{}です
    println!("With text:\n{}", contents);
}
```

> rust bookとは全く違うコードです。
>
> この辺りは以下の動画を参考にしています。

[参考動画](https://www.youtube.com/watch?v=XYkiwsplDTg&list=PLai5B987bZ9CoVR-QEIN9foz4QCJ0H2Y8&index=15)

リスト12-4: 第2引数で指定されたファイルの中身を読み込む

最初に、もう何個かuse文を追記して、標準ライブラリの関係のある箇所を持ってきています: ファイルを扱うのにstd::fs::Fileが必要ですし、 std::io::prelude::*はファイル入出力を含む入出力処理をするのに有用なトレイトを色々含んでいます。 言語が一般的な初期化処理で特定の型や関数を自動的にスコープに導入するように、 std::ioモジュールにはそれ独自の共通の型や関数の初期化処理があり、入出力を行う際に必要になるわけです。 標準の初期化処理とは異なり、std::ioの初期化処理には明示的にuse文を加えなければなりません。

mainに3文を追記しました: 一つ目が、File::open関数を呼んでfilename変数の値に渡して、 ファイルへの可変なハンドルを得る処理です。二つ目が、contentsという名の変数を生成して、 可変で空のStringを割り当てる処理です。この変数が、ファイル読み込み後に中身を保持します。 三つ目が、ファイルハンドルに対してread_to_stringを呼び出し、引数としてcontentsへの可変参照を渡す処理です。

それらの行の後に、今回もファイル読み込み後にcontentsの値を出力する一時的なprintln!文を追記したので、 ここまでプログラムがきちんと動作していることを確認できます。

第1コマンドライン引数には適当な文字列(まだ検索する箇所は実装してませんからね)を、第2引数にpoem.txtファイルを入れて、 このコードを実行しましょう:

```
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

素晴らしい！コードがファイルの中身を読み込み、出力するようになりました。しかし、このコードにはいくつか欠陥があります。 main関数が複数の責任を受け持っています: 一般に、各関数がただ一つの責任だけを持つようになれば、 関数は明確かつ、管理しやすくなります。もう一つの問題点は、できうる限りのエラー処理を怠っていることです。 まだプログラムが小規模なので、これらの欠陥は大きな問題にはなりませんが、プログラムが大規模になるにつれ、 それを綺麗に解消するのは困難になっていきます。プログラムを開発する際に早い段階でリファクタリングを行うのは、 良い戦術です。リファクタリングするコードの量が少なければ、はるかに簡単になりますからね。次は、それを行いましょう。

> 次はリファクタリングです。
>
> この辺りは結構難しいですが、3回ぐらいやっていくとすごくわかりやすいです。