# テストの実行のされ方を制御する
cargo runがコードをコンパイルし、出来上がったバイナリを走らせるのと全く同様に、 cargo testはコードをテストモードでコンパイルし、出来上がったテストバイナリを実行します。 コマンドラインオプションを指定してcargo testの既定動作を変更することができます。 例えば、cargo testで生成されるバイナリの既定動作は、テストを全て並行に実行し、 テスト実行中に生成された出力をキャプチャして出力が表示されるのを防ぎ、 テスト結果に関係する出力を読みやすくすることです。

コマンドラインオプションの中にはcargo testにかかるものや、出来上がったテストバイナリにかかるものがあります。 この2種の引数を区別するために、cargo testにかかる引数を--という区分記号の後に列挙し、 それからテストバイナリにかかる引数を列挙します。cargo test --helpを走らせると、cargo testで使用できるオプションが表示され、 cargo test -- --helpを走らせると、--という区分記号の後に使えるオプションが表示されます。

## テストを並行または連続して実行する
複数のテストを実行するとき、標準では、スレッドを使用して並行に走ります。これはつまり、 テストが早く実行し終わり、コードが機能しているいかんにかかわらず、反応をより早く得られることを意味します。 テストは同時に実行されているので、テストが相互や共有された環境を含む他の共通の状態に依存してないことを確かめてください。 現在の作業対象ディレクトリや環境変数などですね。

例えば、各テストがディスクにtest_output.txtというファイルを作成し、何らかのデータを書き込むコードを走らせるとしてください。 そして、各テストはそのファイルのデータを読み取り、ファイルが特定の値を含んでいるとアサーションし、 その値は各テストで異なります。テストが同時に走るので、あるテストが、 他のテストが書き込んだり読み込んだりする間隙にファイルを上書きするかもしれません。 それから2番目のテストが失敗します。コードが不正だからではなく、 並行に実行されている間にテストがお互いに邪魔をしてしまったせいです。 各テストが異なるファイルに書き込むことを確かめるのが一つの解決策です; 別の解決策では、 一度に一つのテストを実行します。

並行にテストを実行したくなかったり、使用されるスレッド数をよりきめ細かく制御したい場合、 --test-threadsフラグと使用したいスレッド数をテストバイナリに送ることができます。 以下の例に目を向けてください:

```
$ cargo test -- --test-threads=1
```

テストスレッドの数を1にセットし、並行性を使用しないようにプログラムに指示しています。 1スレッドのみを使用してテストを実行すると、並行に実行するより時間がかかりますが、 状態を共有していても、お互いに邪魔をすることはありません。

> テストをするときにスレッドの数を制御することによって依存関係のあるコードもテストができるが、時間はかかりますよという形です。
>
> これは色々やってみないとわからないですね。

## 関数の出力を表示する

標準では、テストが通ると、Rustのテストライブラリは標準出力に出力されたものを全てキャプチャします。例えば、 テストでprintln!を呼び出してテストが通ると、println!の出力は、端末に表示されません; テストが通ったことを示す行しか見られないでしょう。テストが失敗すれば、 残りの失敗メッセージと共に、標準出力に出力されたものが全て見えるでしょう。

例として、リスト11-10は引数の値を出力し、10を返す馬鹿げた関数と通過するテスト1つ、失敗するテスト1つです。

ファイル名: src/lib.rs

```rust
#![allow(unused)]
fn main() {
fn prints_and_returns_10(a: i32) -> i32 {
    //{}という値を得た
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
}
```

リスト11-10: println!を呼び出す関数用のテスト

これらのテストをcargo testで実行すると、以下のような出力を目の当たりにするでしょう:

```
running 2 tests
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

failures:

---- tests::this_test_will_fail stdout ----
        I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

この出力のどこにも I got the value 4 と表示されていないことに注意してください。 これは、テストに合格した場合に出力されるものです。 その出力はキャプチャされてしまいました。 失敗したテストのからの出力 I got the value 8 がテストサマリー出力のセクションに表示され、テストが失敗した原因も示されます。

通過するテストについても出力される値が見たかったら、出力キャプチャ機能を--nocaptureフラグで無効化することができます:

```
$ cargo test -- --nocapture
```

リスト11-10のテストを--nocaptureフラグと共に再度実行したら、以下のような出力を目の当たりにします:

```
running 2 tests
I got the value 4
I got the value 8
test tests::this_test_will_pass ... ok
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
test tests::this_test_will_fail ... FAILED

failures:

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

テスト用の出力とテスト結果の出力がまぜこぜになっていることに注意してください; その理由は、前節で語ったようにテストが並行に実行されているからです。 -test-threads=1オプションと--nocaptureフラグを使ってみて、 その時、出力がどうなるか確かめてください！

## 名前でテストの一部を実行する
時々、全テストを実行すると時間がかかってしまうことがあります。特定の部分のコードしか対象にしていない場合、 そのコードに関わるテストのみを走らせたいかもしれません。cargo testに走らせたいテストの名前を引数として渡すことで、 実行するテストを選ぶことができます。

テストの一部を走らせる方法を模擬するために、リスト11-11に示したように、 add_two関数用に3つテストを作成し、走らせるテストを選択します。

ファイル名: src/lib.rs

```rust
#![allow(unused)]
fn main() {
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
}
```

リスト11-11: 異なる名前の3つのテスト

以前見かけたように、引数なしでテストを走らせたら、全テストが並行に走ります:

```
running 3 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## 単独のテストを走らせる
あらゆるテスト関数の名前をcargo testに渡して、そのテストのみを実行することができます:

> 一つのテストコードだけ走らせたい場合もできます。

```
$ cargo test one_hundred
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out
```

one_hundredという名前のテストだけが走りました; 他の2つのテストはその名前に合致しなかったのです。 まとめ行の最後に2 filtered outと表示することでテスト出力は、このコマンドが走らせた以上のテストがあることを知らせてくれています。

この方法では、複数のテストの名前を指定することはできません; cargo testに与えられた最初の値のみが使われるのです。 ですが、複数のテストを走らせる方法もあります。

## 複数のテストを実行するようフィルターをかける
テスト名の一部を指定でき、その値に合致するあらゆるテストが走ります。例えば、 我々のテストの2つがaddという名前を含むので、cargo test addを実行することで、その二つを走らせることができます:

```
$ cargo test add
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 2 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

このコマンドは名前にaddを含むテストを全て実行し、one_hundredという名前のテストを除外しました。 また、テストが出現するモジュールがテスト名の一部になっていて、 モジュール名でフィルターをかけることで、あるモジュール内のテスト全てを実行できることに注目してください。

## 特に要望のない限りテストを無視する
時として、いくつかの特定のテストが実行するのに非常に時間がかかることがあり、 cargo testの実行のほとんどで除外したくなるかもしれません。引数として確かに実行したいテストを全て列挙するのではなく、 ここに示したように代わりに時間のかかるテストをignore属性で除外すると注釈することができます。

ファイル名: src/lib.rs
``` rust
#![allow(unused)]
fn main() {
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // 実行に1時間かかるコード
    // code that takes an hour to run
}
}
```

#[test]の後の除外したいテストに#[ignore]行を追加しています。これで、 テストを実行したら、it_worksは実行されるものの、expensive_testは実行されません:

> テスト全体で無視することもできます。

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out
```

expensive_test関数は、ignoredと列挙されています。無視されるテストのみを実行したかったら、 cargo test -- --ignoredを使うことができます:

```
$ cargo test -- --ignored
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

どのテストを走らせるか制御することで、結果が早く出ることを確かめることができるのです。 ignoredテストの結果を確認することが道理に合い、結果を待つだけの時間ができたときに、 代わりにcargo test -- --ignoredを走らせることができます。

> testコードに関しては色々なテストができます。戦略的に考えて色々やっていきましょう！