# テスト駆動開発でライブラリの機能を開発する
今や、ロジックをsrc/lib.rsに抜き出し、引数集めとエラー処理をsrc/main.rsに残したので、 コードの核となる機能のテストを書くのが非常に容易になりました。いろんな引数で関数を直接呼び出し、 コマンドラインからバイナリを呼び出す必要なく戻り値を確認できます。ご自由にConfig::newやrun関数の機能のテストは、 ご自身でお書きください。

この節では、テスト駆動開発(TDD)過程を活用してminigrepプログラムに検索ロジックを追加します。 このソフトウェア開発テクニックは、以下の手順に従います:

1. 失敗するテストを書き、走らせて想定通りの理由で失敗することを確かめる。
2. 十分な量のコードを書くか変更して新しいテストを通過するようにする。
3. 追加または変更したばかりのコードをリファクタリングし、テストが通り続けることを確認する。
4. 手順1から繰り返す！

この過程は、ソフトウェアを書く多くの方法のうちの一つに過ぎませんが、TDDによりコードデザインも駆動することができます。 テストを通過させるコードを書く前にテストを書くことで、過程を通して高いテストカバー率を保つ助けになります。

実際にクエリ文字列の検索を行う機能の実装をテスト駆動し、クエリに合致する行のリストを生成します。 この機能をsearchという関数に追加しましょう。

## 失敗するテストを記述する
もう必要ないので、プログラムの振る舞いを確認していたprintln!文をsrc/lib.rsとsrc/main.rsから削除しましょう。 それからsrc/lib.rsで、テスト関数のあるtestモジュールを追加します。第11章のようにですね。 このテスト関数がsearch関数に欲しい振る舞いを指定します: クエリとそれを検索するテキストを受け取り、 クエリを含む行だけをテキストから返します。リスト12-15にこのテストを示していますが、まだコンパイルは通りません。

ファイル名: src/lib.rs

```rust
#![allow(unused)]
fn main() {
fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
     vec![]
}

#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        // Rustは
        // 安全で速く生産性も高い。
        // 3つ選んで。
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }
}
}
```

リスト12-15: こうだったらいいなというsearch関数の失敗するテストを作成する

このテストは、"duct"という文字列を検索します。検索対象の文字列は3行で、うち1行だけが"duct"を含みます。 search関数から返る値が想定している行だけを含むことをアサーションします。

このテストを走らせ、失敗するところを観察することはできません。このテストはコンパイルもできないからです: まだsearch関数が存在していません！ゆえに今度は、空のベクタを常に返すsearch関数の定義を追加することで、 テストをコンパイルし走らせるだけのコードを追記します。リスト12-16に示したようにですね。そうすれば、 テストはコンパイルでき、失敗するはずです。なぜなら、空のベクタは、 "safe, fast, productive."という行を含むベクタとは合致しないからです。

> こうだったらいいなというsearch関数の失敗するテストを作成する
>
> マジでこの表現

ファイル名: src/lib.rs

``` rust
#![allow(unused)]
fn main() {
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
}
}
```

これライフタイムとかやvecの返り値はしっかり定義する必要があるよ。

リスト12-16: テストがコンパイルできるのに十分なだけsearch関数を定義する

明示的なライフタイムの'aがsearchのシグニチャで定義され、contents引数と戻り値で使用されていることに注目してください。 第10章からライフタイム仮引数は、どの実引数のライフタイムが戻り値のライフタイムに関連づけられているかを指定することを思い出してください。 この場合、返却されるベクタは、 (query引数ではなく)contents引数のスライスを参照する文字列スライスを含むべきと示唆しています。

言い換えると、コンパイラにsearch関数に返されるデータは、 search関数にcontents引数で渡されているデータと同期間生きることを教えています。 これは重要なことです！スライスに参照されるデータは、参照が有効になるために有効である必要があるのです; コンパイラがcontentsではなくqueryの文字列スライスを生成すると想定してしまったら、 安全性チェックを間違って行うことになってしまいます。

ライフタイム注釈を忘れてこの関数をコンパイルしようとすると、こんなエラーが出ます:

```
error[E0106]: missing lifetime specifier
(エラー: ライフタイム指定子が欠けています)
 --> src/lib.rs:5:51
  |
5 | pub fn search(query: &str, contents: &str) -> Vec<&str> {
  |                                                   ^ expected lifetime
parameter
  |
  = help: this function's return type contains a borrowed value, but the
  signature does not say whether it is borrowed from `query` or `contents`
  (助言: この関数の戻り値は、借用された値を含んでいますが、シグニチャにはそれが、
  `query`か`contents`から借用されたものであるかが示されていません)
```

これはqueryもcontentsも同じ型情報やからどっちを返すの？とわからないためにライフタイムの宣言をすることによって返り値の中に含まれるものがなんなのか？明確にできます。

コンパイラには、二つの引数のどちらが必要なのか知る由がないので、教えてあげる必要があるのです。 contentsがテキストを全て含む引数で、合致するそのテキストの一部を返したいので、 contentsがライフタイム記法で戻り値に関連づくはずの引数であることをプログラマは知っています。

他のプログラミング言語では、シグニチャで引数と戻り値を関連づける必要はありません。これは奇妙に思えるかもしれませんが、 時間とともに楽になっていきます。この例を第10章、「ライフタイムで参照を有効化する」節と比較したくなるかもしれません。

さあ、テストを実行しましょう:

```
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
--warnings--
    Finished dev [unoptimized + debuginfo] target(s) in 0.43 secs
     Running target/debug/deps/minigrep-abcabcabc

running 1 test
test test::one_result ... FAILED

failures:

---- test::one_result stdout ----
        thread 'test::one_result' panicked at 'assertion failed: `(left ==
right)`
left: `["safe, fast, productive."]`,
right: `[]`)', src/lib.rs:48:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.


failures:
    test::one_result

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

素晴らしい。テストは全く想定通りに失敗しています。テストが通るようにしましょう！

## テストを通過させるコードを書く
空のベクタを常に返しているために、現状テストは失敗しています。それを修正し、searchを実装するには、 プログラムは以下の手順に従う必要があります:

- 中身を各行ごとに繰り返す。
- 行にクエリ文字列が含まれるか確認する。
- するなら、それを返却する値のリストに追加する。
- しないなら、何もしない。
- 一致する結果のリストを返す。
- 各行を繰り返す作業から、この手順に順に取り掛かりましょう。

### linesメソッドで各行を繰り返す
Rustには、文字列を行ごとに繰り返す役立つメソッドがあり、利便性のためにlinesと名付けられ、 リスト12-17のように動作します。まだ、これはコンパイルできないことに注意してください。

ファイル名: src/lib.rs

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        // 行に対して何かする
        // do something with line
    }
}
```

リスト12-17: contentsの各行を繰り返す

linesメソッドはイテレータを返します。イテレータについて詳しくは、第13章で話しますが、 リスト3-5でこのようなイテレータの使用法は見かけたことを思い出してください。 そこでは、イテレータにforループを使用してコレクションの各要素に対して何らかのコードを走らせていました。

### クエリを求めて各行を検索する
次に現在の行がクエリ文字列を含むか確認します。幸運なことに、 文字列にはこれを行ってくれるcontainsという役に立つメソッドがあります！search関数に、 containsメソッドの呼び出しを追加してください。リスト12-18のようにですね。 それでもまだコンパイルできないことに注意してください。

ファイル名: src/lib.rs

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

リスト12-18: 行がqueryの文字列を含むか確認する機能を追加する

### 合致した行を保存する
また、クエリ文字列を含む行を保存する方法が必要です。そのために、forループの前に可変なベクタを生成し、 pushメソッドを呼び出してlineをベクタに保存することができます。forループの後でベクタを返却します。 リスト12-19のようにですね。

ファイル名: src/lib.rs

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

リスト12-19: 合致する行を保存したので、返すことができる

これでsearch関数は、queryを含む行だけを返すはずであり、テストも通るはずです。 テストを実行しましょう:

```
$ cargo test
--snip--
running 1 test
test test::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

テストが通り、動いていることがわかりました！

ここで、テストが通過するよう保ったまま、同じ機能を保持しながら、検索関数の実装をリファクタリングする機会を考えることもできます。 検索関数のコードは悪すぎるわけではありませんが、イテレータの有用な機能の一部を活用していません。 この例には第13章で再度触れ、そこでは、イテレータをより深く探究し、さらに改善する方法に目を向けます。

### run関数内でsearch関数を使用する
search関数が動きテストできたので、run関数からsearchを呼び出す必要があります。config.queryの値と、 ファイルからrunが読み込むcontentsの値をsearch関数に渡す必要があります。 それからrunは、searchから返ってきた各行を出力するでしょう:

ファイル名: src/lib.rs

```rust
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    for line in search(&config.query, &contents) {
        println!("{}", line);
    }

    Ok(())
}
```

それでもforループでsearchから各行を返し、出力しています。

さて、プログラム全体が動くはずです！試してみましょう。まずはエミリー・ディキンソンの詩から、 ちょうど1行だけを返すはずの言葉から。"frog"です:

```
$ cargo run frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38 secs
     Running `target/debug/minigrep frog poem.txt`
How public, like a frog
```

かっこいい！今度は、複数行にマッチするであろう言葉を試しましょう。"body"とかね:

```
$ cargo run body poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep body poem.txt`
I’m nobody! Who are you?
Are you nobody, too?
How dreary to be somebody!
```

そして最後に、詩のどこにも現れない単語を探したときに、何も出力がないことを確かめましょう。 "monomorphization"などね:

```
$ cargo run monomorphization poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep monomorphization poem.txt`
```

最高です！古典的なツールの独自のミニバージョンを構築し、アプリケーションを構造化する方法を多く学びました。 また、ファイル入出力、ライフタイム、テスト、コマンドライン引数の解析についても、少し学びました。

このプロジェクトをまとめ上げるために、環境変数を扱う方法と標準エラー出力に出力する方法を少しだけデモします。 これらはどちらも、コマンドラインプログラムを書く際に有用です。

> こういった感じのテスト駆動開発を訓練によって磨かれます。