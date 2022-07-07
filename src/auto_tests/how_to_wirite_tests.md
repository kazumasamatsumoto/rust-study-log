# テストの記述法
テストは、テスト以外のコードが想定された方法で機能していることを実証するRustの関数です。 テスト関数の本体は、典型的には以下の3つの動作を行います:

1. 必要なデータや状態をセットアップする。
2. テスト対象のコードを走らせる。
3. 結果が想定通りであることを断定（以下、アサーションという）する。

Rustが、特にこれらの動作を行うテストを書くために用意している機能を見ていきましょう。 これには、test属性、いくつかのマクロ、should_panic属性が含まれます。

## テスト関数の構成
最も単純には、Rustにおけるテストはtest属性で注釈された関数のことです。属性とは、 Rustコードの部品に関するメタデータです; 一例を挙げれば、構造体とともに第5章で使用したderive属性です。 関数をテスト関数に変えるには、fnの前に#[test]を付け加えてください。 cargo testコマンドでテストを実行したら、コンパイラはtest属性で注釈された関数を走らせるテスト用バイナリをビルドし、 各テスト関数が通過したか失敗したかを報告します。

> fnの前に#[test]をつける
>
> cargo testでテストを実行する

新しいライブラリプロジェクトをCargoで作ると、テスト関数付きのテストモジュールが自動的に生成されます。 このモジュールのおかげで、新しいプロジェクトを始めるたびにテスト関数の正しい構造とか文法をいちいち検索しなくてすみます。 ここに好きな数だけテスト関数やテストモジュールを追加すればいいというわけです！

> 新規作成のでは意識しなくていい

まず、実際にはコードをテストしない、自動生成されたテンプレートのテストで実験して、テストの動作の性質をいくらか学びましょう。 その後で、以前書いたコードを呼び出し、振る舞いが正しいことをアサーションする、ホンモノのテストを書きましょう。

adderという新しいライブラリプロジェクトを生成しましょう:

```
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

adderライブラリのsrc/lib.rsファイルの中身は、リスト11-1のような見た目のはずです。

ファイル名: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

リスト11-1: cargo newで自動生成されたテストモジュールと関数

とりあえず、最初の2行は無視し、関数に集中してその動作法を見ましょう。 fn行の#[test]注釈に注目してください: この属性は、これがテスト関数であることを示すので、 テスト実行機はこの関数をテストとして扱うとわかるのです。さらに、testsモジュール内にはテスト関数以外の関数を入れ、 一般的なシナリオをセットアップしたり、共通の処理を行う手助けをしたりもできるので、 #[test]属性でどの関数がテストかを示す必要があるのです。

関数本体は、assert_eq!マクロを使用して、2 + 2が4に等しいことをアサーションしています。 このアサーションは、典型的なテストのフォーマット例をなしているわけです。走らせてこのテストが通る（訳注：テストが成功する、の意味。英語でpassということから、このように表現される）ことを確かめましょう。

> 要するにこのテストでは２＋２は４ですか？というテストをしています。
> この感覚ですが、僕も最初は「は？」という感覚がありましたが、これは我々が小学生の時から計算を日頃から実施しているので当然のごとくかと思います。
> 重要なのはA＝Bの式が成り立つのか？そういった観点がテストとして重要です。
> もう一度言いますA=Bの条件が成り立つのか？テストはこのAとBの項目を作り出すのがややこしくなるので複雑になります。
> 別々のアプローチで導き出された項目が両者あっているからテストに合格するという流れの認識です。これがないと結構苦労しました。（てか僕が苦労しました。）

cargo testコマンドでプロジェクトにあるテストが全て実行されます。リスト11-2に示したようにですね。

```
adder$ cargo test
   Compiling adder v0.1.0 (/Users/matsumotokazumasa/poc/rust/rust-programing/adder)
    Finished test [unoptimized + debuginfo] target(s) in 2.22s
     Running unittests src/lib.rs (target/debug/deps/adder-d053b13dba04ef5b)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

adder$
```

リスト11-2: 自動生成されたテストを走らせた出力

Cargoがテストをコンパイルし、走らせました。Compiling, Finished, Runningの行の後にrunning 1 testの行があります。 次行が、生成されたテスト関数のit_worksという名前とこのテストの実行結果、okを示しています。 テスト実行の総合的なまとめが次に出現します。test result:ok.というテキストは、 全テストが通ったことを意味し、1 passed; 0 failedと読める部分は、通過または失敗したテストの数を合計しているのです。

> 順番は
>
> ・Compiling（コンパイル）
>
> ・Finished（コンパイル完了）
>
> ・Running（実行中）
>
> ・動作確認結果
>
> ・実行結果
>
> ・テスト結果のまとめ
>
> ・テスト数とテストが通った数と失敗した数の表示
>
> という順番に表示されます。

無視すると指定したテストは何もなかったため、まとめは0 ignoredと示しています。 また、実行するテストにフィルタをかけもしなかったので、まとめの最後に0 filtered outと表示されています。 テストを無視することとフィルタすることに関しては次の節、テストの実行され方を制御するで語ります。

> テストは「無視すること」と「フィルタリングすること」ができる

0 measuredという統計は、パフォーマンスを測定するベンチマークテスト用です。 ベンチマークテストは、本書記述の時点では、nightly版のRustでのみ利用可能です。 詳しくは、ベンチマークテストのドキュメンテーションを参照されたし。

[ベンチマークテストとは？](https://doc.rust-lang.org/unstable-book/library-features/test.html)

テスト出力の次の部分、つまりDoc-tests adderで始まる部分は、ドキュメンテーションテストの結果用のものです。 まだドキュメンテーションテストは何もないものの、コンパイラは、APIドキュメントに現れるどんなコード例もコンパイルできます。 この機能により、ドキュメントとコードを同期することができるわけです。ドキュメンテーションテストの書き方については、 第14章のテストとしてのドキュメンテーションコメント節で議論しましょう。今は、Doc-tests出力は無視します。

テストの名前を変更してどうテスト出力が変わるか確かめましょう。it_works関数を違う名前、explorationなどに変えてください。 そう、以下のように:

ファイル名: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}

fn main() {}
```

そして、cargo testを再度走らせます。これで出力がit_worksの代わりにexplorationと表示しています:

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.59s
     Running target/debug/deps/adder-92948b65e88960b4

running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

> テストの関数の名前を変えるとテストの出力の表示も少し変わっていきます。

別のテストを追加しますが、今回は失敗するテストにしましょう！テスト関数内の何かがパニックすると、 テストは失敗します。各テストは、新規スレッドで実行され、メインスレッドが、テストスレッドが死んだと確認した時、 テストは失敗と印づけられます。第9章でパニックを引き起こす最も単純な方法について語りました。 そう、panic!マクロを呼び出すことですね。src/lib.rsファイルがリスト11-3のような見た目になるよう、 新しいテストanotherを入力してください。

ファイル名: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        //このテストを失敗させる
        panic!("Make this test fail");
    }
}

fn main() {}
```

リスト11-3: panic!マクロを呼び出したために失敗する2番目のテストを追加する
> このテストは必ず失敗します。

cargo testで再度テストを走らせてください。出力はリスト11-4のようになるはずであり、 explorationテストは通り、anotherは失敗したと表示されます。

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.72s
     Running target/debug/deps/adder-92948b65e88960b4

running 2 tests
test tests::another ... FAILED
test tests::exploration ... ok

failures:

---- tests::another stdout ----
thread 'main' panicked at 'Make this test fail', src/lib.rs:10:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

リスト11-4: 一つのテストが通り、一つが失敗するときのテスト結果

okの代わりにtest test::anotherの行は、FAILEDを表示しています。個々の結果とまとめの間に、 2つ新たな区域ができました: 最初の区域は、失敗したテスト各々の具体的な理由を表示しています。 今回の場合、anotherは'Make this test fail'でパニックしたために失敗し、 これは、src/lib.rsファイルの10行で起きました。次の区域は失敗したテストの名前だけを列挙しています。 これは、テストがたくさんあり、失敗したテストの詳細がたくさん表示されるときに有用になります。 失敗したテストの名前を使用してそのテストだけを実行し、より簡単にデバッグすることができます。 テストの実行方法については、テストの実行され方を制御する節でもっと語りましょう。

サマリー行が最後に出力されています: 総合的に言うと、テスト結果はFAILEDでした。 一つのテストが通り、一つが失敗したわけです。

様々な状況でのテスト結果がどんな風になるか見てきたので、テストを行う際に有用になるpanic!以外のマクロに目を向けましょう。

## assert!マクロで結果を確認する
assert!マクロは、標準ライブラリで提供されていますが、テスト内の何らかの条件がtrueと評価されることを確かめたいときに有効です。
assert!マクロには、論理値に評価される引数を与えます。その値がtrueなら、 assert!は何もせず、テストは通ります。
その値がfalseなら、assert!マクロはpanic!マクロを呼び出し、 テストは失敗します。
assert!マクロを使用することで、コードが意図した通りに機能していることを確認する助けになるわけです。

第5章のリスト5-15で、Rectangle構造体とcan_holdメソッドを使用しました。
リスト11-5でもそれを繰り返しています。
このコードをsrc/lib.rsファイルに放り込み、assert!マクロでそれ用のテストを何か書いてみましょう。

> trueやfalseでテストを評価する
>
> なので論理値が返ってくるような関数の実行のテストに有用です。

ファイル名: src/lib.rs

``` rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {}
```

リスト11-5: 第5章からRectangle構造体とそのcan_holdメソッドを使用する

> このメソッドは2つの長方形があり、横と縦の長さがどちらも大きいみたいな感じのイメージになります。

![メソッド](../imgs/auto_tests/rectangle.png)

can_holdメソッドは論理値を返すので、assert!マクロの完璧なユースケースになるわけです。 リスト11-6で、幅が8、高さが7のRectangleインスタンスを生成し、これが幅5、 高さ1の別のRectangleインスタンスを保持できるとアサーションすることでcan_holdを用いるテストを書きます。

ファイル名: src/lib.rs

``` rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}

fn main() {}
```

リスト11-6: より大きな長方形がより小さな長方形を確かに保持できるかを確認するcan_hold用のテスト

testsモジュール内に新しい行を加えたことに注目してください: use super::*です。 testsモジュールは、第7章のモジュールツリーの要素を示すためのパス節で講義した通常の公開ルールに従う普通のモジュールです。 testsモジュールは、内部モジュールなので、外部モジュール内のテスト配下にあるコードを内部モジュールのスコープに持っていく必要があります。 ここではglobを使用して、外部モジュールで定義したもの全てがこのtestsモジュールでも使用可能になるようにしています。

テストはlarger_can_hold_smallerと名付け、必要なRectangleインスタンスを2つ生成しています。 そして、assert!マクロを呼び出し、larger.can_hold(&smaller)の呼び出し結果を渡しました。 この式は、trueを返すと考えられるので、テストは通るはずです。確かめましょう！

```
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running target/debug/deps/rectangle-6584c4561e48942e

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

通ります！別のテストを追加しましょう。今回は、小さい長方形は、より大きな長方形を保持できないことをアサーションします。

ファイル名: src/lib.rs

``` rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(!smaller.can_hold(&larger));
    }
}

fn main() {}
```

今回の場合、can_hold関数の正しい結果はfalseなので、その結果をassert!マクロに渡す前に反転させる必要があります。 結果として、can_holdがfalseを返せば、テストは通ります。

> 結果を反転させたい時は「!」を設定する必要があるassert(!smaller.can_hold(&larger))みたいな感じです。

```
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running target/debug/deps/rectangle-6584c4561e48942e

running 2 tests
test tests::larger_can_hold_smaller ... ok
test tests::smaller_cannot_hold_larger ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

通るテストが2つ！さて、コードにバグを導入したらテスト結果がどうなるか確認してみましょう。 幅を比較する大なり記号を小なり記号で置き換えてcan_holdメソッドの実装を変更しましょう:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

// ここの記号を変更させる
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width < other.width && self.height > other.height
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(!smaller.can_hold(&larger));
    }
}

fn main() {}
```

テストを実行すると、以下のような出力をします

```
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running target/debug/deps/rectangle-6584c4561e48942e

running 2 tests
test tests::larger_can_hold_smaller ... FAILED
test tests::smaller_cannot_hold_larger ... ok

failures:

---- tests::larger_can_hold_smaller stdout ----
thread 'main' panicked at 'assertion failed: larger.can_hold(&smaller)', src/lib.rs:28:9
(スレッド'main'はsrc/lib.rs:28:9の'assertion failed: larger.can_hold(&smaller)'でパニックしました)
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.

failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

テストによりバグが捕捉されました！larger.widthが8、smaller.widthが5なので、 can_hold内の幅の比較が今はfalseを返すようになったのです: 8は5より小さくないですからね。

## assert_eq!とassert_ne!マクロで等値性をテストする
機能をテストする一般的な方法は、テスト下にあるコードの結果をコードが返すと期待される値と比較して、 等しいと確かめることです。これをassertマクロを使用して==演算子を使用した式を渡すことで行うこともできます。 しかしながら、これはありふれたテストなので、標準ライブラリには1組のマクロ(assert_eq!とassert_ne!)が提供され、 このテストをより便利に行うことができます。これらのマクロはそれぞれ、二つの引数を比べ、等しいかと等しくないかを確かめます。 また、アサーションが失敗したら二つの値の出力もし、テストが失敗した原因を確認しやすくなります。 一方でassert!マクロは、==式の値がfalseになったことしか示さず、falseになった原因の値は出力しません。

リスト11-7において、引数に2を加えて結果を返すadd_twoという名前の関数を書いています。 そして、assert_eq!マクロでこの関数をテストしています。

> 返り値が数値や文字列スライスを返すような関数では等値性（値が一緒かどうか？）をテストする

ファイル名: src/lib.rs

``` rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}

fn main() {}
```
リスト11-7: assert_eq!マクロでadd_two関数をテストする

> これだと、左がテストの答えの結果、右が関数の結果です。これは左と右をどちらでも同じです。

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running target/debug/deps/adder-92948b65e88960b4

running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

assert_eq!マクロに与えた第1引数の4は、add_two(2)の呼び出し結果と等しいです。 このテストの行はtest tests::it_adds_two ... okであり、okというテキストはテストが通ったことを示しています！

コードにバグを仕込んで、assert_eq!を使ったテストが失敗した時にどんな見た目になるのか確認してみましょう。 add_two関数の実装を代わりに3を足すように変えてください:

This code does not produce the desired behavior.

```rust
pub fn add_two(a: i32) -> i32 {
    a + 3
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}

fn main() {}
```

テストを再度実行します:

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running target/debug/deps/adder-92948b65e88960b4

running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.


failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

テストがバグを捕捉しました！it_adds_twoのテストは失敗し、assertion failed: `(left == right)`というメッセージを表示し、 leftは4で、rightは5だったと示しています。このメッセージは有用で、デバッグを開始する助けになります: assert_eq!のleft引数は4だったが、add_two(2)があるright引数は5だったことを意味しています。

二つの値が等しいとアサーションを行う関数の引数を expectedとactualと呼び、引数を指定する順序が問題になる言語やテストフレームワークもあることに注意してください。 ですがRustでは、leftとrightと呼ばれ、期待する値とテスト下のコードが生成する値を指定する順序は 問題になりません。今回のテストのアサーションをassert_eq!(add_two(2), 4)と書くこともでき、 そうすると失敗メッセージは、assertion failed: `(left == right)`となり、 leftが5でrightが4と表示されるでしょう。

assert_ne!マクロは、与えた2つの値が等しくなければ通り、等しければ失敗します。 このマクロは、値が何になるだろうか確信が持てないけれども、コードが意図した通りに動いていれば、 確実にこの値にはならないだろうとわかっているような場合に最も有用になります。例えば、 入力を何らかの手段で変え（て出力す）ることが保証されているけれども、入力の変え方がテストを実行する曜日に依存する関数をテストしているなら、 アサーションすべき最善の事柄は、関数の出力が入力と等しくないことかもしれません。

内部的には、assert_eq!とassert_ne!マクロは、それぞれ==と!=演算子を使用しています。 アサーションが失敗すると、これらのマクロは引数をデバッグフォーマットを使用してプリントするので、 比較対象の値はPartialEqとDebugトレイトを実装していなければなりません。 すべての組み込み型と、ほぼすべての標準ライブラリの型はこれらのトレイトを実装しています。 自分で定義した構造体やenumについては、 その型の値が等しいか等しくないかをアサーションするために、PartialEqを実装する必要があるでしょう。 それが失敗した時にその値をプリントできるように、Debugを実装する必要もあるでしょう。 第5章のリスト5-12で触れたように、どちらのトレイトも導出可能なトレイトなので、 これは通常、単純に構造体やenum定義に#[derive(PartialEq, Debug)]という注釈を追加するだけですみます。 これらやその他の導出可能なトレイトに関する詳細については、付録C、導出可能なトレイトをご覧ください。

## カスタムの失敗メッセージを追加する
さらに、assert!、assert_eq!、assert_ne!の追加引数として、失敗メッセージと共にカスタムのメッセージが表示されるよう、 追加することもできます。assert!の1つの必須引数の後に、 あるいはassert_eq!とassert_ne!の2つの必須引数の後に指定された引数はすべてformat!マクロに渡されるので、 （format!マクロについては第8章の+演算子、またはformat!マクロで連結節で議論しました）、 {}プレースホルダーを含むフォーマット文字列とこのプレースホルダーに置き換えられる値を渡すことができます。 カスタムメッセージは、アサーションがどんな意味を持つかドキュメント化するのに役に立ちます; もしテストが失敗した時、コードにどんな問題があるのかをよりしっかり把握できるはずです。

> エラーメッセージを追加できると「わかりにくさ」から「わかりやすさ」が表現されます
>
> 例えば日本語のエラーメッセージは表現がしやすいです。

例として、人々に名前で挨拶をする関数があり、関数に渡した名前が出力に出現することをテストしたいとしましょう:

ファイル名: src/lib.rs

``` rust
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}

fn main() {}
```

このプログラムの要件はまだ取り決められておらず、挨拶の先頭のHelloというテキストはおそらく変わります。 要件が変わった時にテストを更新しなくてもよいようにしたいと考え、 greeting関数から返る値と正確な等値性を確認するのではなく、出力が入力引数のテキストを含むことをアサーションするだけにします。

greetingがnameを含まないように変更してこのコードにバグを仕込み、このテストの失敗がどんな風になるのか確かめましょう:

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}

fn main() {}
```

このテストを実行すると、以下のように出力されます:

```
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running target/debug/deps/greeter-170b942eb5bf5e3a

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'main' panicked at 'assertion failed: result.contains("Carol")', src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

この結果は、アサーションが失敗し、どの行にアサーションがあるかを示しているだけです。 今回の場合、失敗メッセージがgreeting関数から得た値を出力していればより有用でしょう。 テスト関数を変更し、 greeting関数から得た実際の値で埋められるプレースホルダーを含むフォーマット文字列からなるカスタムの失敗メッセージを与えてみましょう。

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            //挨拶(greeting)は名前を含んでいません。その値は`{}`でした
            "Greeting did not contain name, value was `{}`",
            result
        );
    }
}
```

これでテストを実行したら、より有益なエラーメッセージが得られるでしょう:

```
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.93s
     Running target/debug/deps/greeter-170b942eb5bf5e3a

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'main' panicked at 'Greeting did not contain name, value was `Hello!`', src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

実際に得られた値がテスト出力に表示されているので、起こると想定していたものではなく、 起こったものをデバッグするのに役に立ちます。

## should_panicでパニックを確認する
期待する正しい値をコードが返すことを確認することに加えて、想定通りにコードがエラー状態を扱っていることを確認するのも重要です。 例えば、第9章のリスト9-10で生成したGuess型を考えてください。Guessを使用する他のコードは、 Guessのインスタンスは1から100の範囲の値しか含まないという保証に依存しています。 その範囲外の値でGuessインスタンスを生成しようとするとパニックすることを確認するテストを書くことができます。

これは、テスト関数にshould_panicという別の属性を追加することで達成できます。 この属性は、関数内のコードがパニックしたら、テストを通過させます。つまり、 関数内のコードがパニックしなかったら、テストは失敗するわけです。

リスト11-8は、予想どおりにGuess::newのエラー条件が発生していることを確認するテストを示しています。

ファイル名: src/lib.rs

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            //予想値は1から100の間でなければなりませんが、{}でした。
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}

fn main() {}
```

リスト11-8: 状況がpanic!を引き起こすとテストする

#[test]属性の後、適用するテスト関数の前に#[should_panic]属性を配置しています。 このテストが通るときの結果を見ましょう:

```
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running target/debug/deps/guessing_game-57d70c3acb738f4d

running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests guessing_game

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

よさそうですね！では、値が100より大きいときにnew関数がパニックするという条件を除去することでコードにバグを導入しましょう:

> 関数の実行結果がパニックになる可能性のある関数を実行した時にパニックすればテスト成功というテストを実施する

``` rust
pub struct Guess {
    value: i32,
}

// --snip--
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            //予想値は1から100の間でなければなりませんが、{}でした。
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}

fn main() {}
```

リスト11-8のテストを実行すると、失敗するでしょう:

```
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.62s
     Running target/debug/deps/guessing_game-57d70c3acb738f4d

running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
note: test did not panic as expected

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

この場合、それほど役に立つメッセージは得られませんが、テスト関数に目を向ければ、 #[should_panic]で注釈されていることがわかります。得られた失敗は、 テスト関数のコードがパニックを引き起こさなかったことを意味するのです。

should_panicを使用するテストは不正確なこともあります。なぜなら、コードが何らかのパニックを起こしたことしか示さないからです。 should_panicのテストは、起きると想定していたもの以外の理由でテストがパニックしても通ってしまうのです。 should_panicのテストの正確を期すために、should_panic属性にexpected引数を追加することもできます。 このテストハーネスは、失敗メッセージに与えられたテキストが含まれていることを確かめてくれます。 例えば、リスト11-9の修正されたGuessのコードを考えてください。ここでは、 new関数は、値が大きすぎるか小さすぎるかによって異なるメッセージでパニックします。

ファイル名: src/lib.rs

```rust
pub struct Guess {
    value: i32,
}

// --snip--
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                //予想値は1以上でなければなりませんが、{}でした。
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                //予想値は100以下でなければなりませんが、{}でした。
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    //予想値は100以下でなければなりません
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}

fn main() {}
```

> パニックが発生したことしかわからないのが欠点でもある

リスト11-9: 状況が特定のパニックメッセージでpanic!を引き起こすことをテストする

should_panic属性のexpected引数に置いた値がGuess::new関数がパニックしたメッセージの一部になっているので、 このテストは通ります。予想されるパニックメッセージ全体を指定することもでき、今回の場合、 Guess value must be less than or equal to 100, got 200.となります。 should_panicの予想される引数に何を指定するかは、パニックメッセージのどこが固有でどこが動的か、 またテストをどの程度正確に行いたいかによります。今回の場合、パニックメッセージの一部でも、テスト関数内のコードが、 else if value > 100の場合を実行していると確認するのに事足りるのです。

expectedメッセージありのshould_panicテストが失敗すると何が起きるのが確かめるために、 if value < 1とelse if value > 100ブロックの本体を入れ替えることで再度コードにバグを仕込みましょう:

> should_panicでパニック時のメッセージでどのようなパニックが発生するかをテストで確認できる。

``` rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                //予想値は100以下でなければなりませんが、{}でした。
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                //予想値は1以上でなければなりませんが、{}でした。
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

should_panicテストを実行すると、今回は失敗するでしょう:

```
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running target/debug/deps/guessing_game-57d70c3acb738f4d

running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
thread 'main' panicked at 'Guess value must be greater than or equal to 1, got 200.', src/lib.rs:13:13
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
note: panic did not contain expected string
      panic message: `"Guess value must be greater than or equal to 1, got 200."`,
 expected substring: `"Guess value must be less than or equal to 100"`

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

この失敗メッセージは、このテストが確かに予想通りパニックしたことを示していますが、 パニックメッセージは、予想される文字列の'Guess value must be less than or equal to 100'を含んでいませんでした。 実際に得られたパニックメッセージは今回の場合、Guess value must be greater than or equal to 1, got 200でした。 そうしてバグの所在地を割り出し始めることができるわけです！

## Result<T, E>をテストで使う
これまでは、失敗するとパニックするようなテストを書いてきましたが、 Result<T, E>を使うようなテストを書くこともできます！ 以下は、Listing 11-1のテストを、Result<T, E>を使い、パニックする代わりにErrを返すように書き直したものです：

```rust
#![allow(unused)]
fn main() {
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
}
```

it_works関数の戻り値の型はResult<(), String>になりました。 関数内でassert_eq!マクロを呼び出す代わりに、テストが成功すればOk(())を、失敗すればErrにStringを入れて返すようにします。

Result<T, E> を返すようなテストを書くと、?演算子をテストの中で使えるようになります。 これは、テスト内で何らかの工程がErrヴァリアントを返したときに失敗するべきテストを書くのに便利です。

Result<T, E>を使うテストに#[should_panic]注釈を使うことはできません。 テストが失敗しなければならないときは、直接Err値を返してください。

今やテスト記法を複数知ったので、テストを走らせる際に起きていることに目を向け、 cargo testで使用できるいろんなオプションを探究しましょう。

> resultのOK、Errを使用してテストをすることも可能です。