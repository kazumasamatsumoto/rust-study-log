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