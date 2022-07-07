# テストの体系化
章の初めで触れたように、テストは複雑な鍛錬であり、人によって専門用語や体系化が異なります。 Rustのコミュニティでは、テストを2つの大きなカテゴリで捉えています: 単体テストと結合テストです。 単体テストは小規模でより集中していて、個別に1回に1モジュールをテストし、非公開のインターフェイスもテストすることがあります。 結合テストは、完全にライブラリ外になり、他の外部コード同様に自分のコードを使用し、公開インターフェイスのみ使用し、 1テストにつき複数のモジュールを用いることもあります。

どちらのテストを書くのも、ライブラリの一部が個別かつ共同でしてほしいことをしていることを確認するのに重要なのです。

## 単体テスト
単体テストの目的は、残りのコードから切り離して各単位のコードをテストし、 コードが想定通り、動いたり動いていなかったりする箇所を迅速に特定することです。 単体テストは、テスト対象となるコードと共に、srcディレクトリの各ファイルに置きます。 慣習は、各ファイルにtestsという名前のモジュールを作り、テスト関数を含ませ、 そのモジュールをcfg(test)で注釈することです。

## テストモジュールと#[cfg(test)]
testsモジュールの#[cfg(test)]という注釈は、コンパイラにcargo buildを走らせた時ではなく、cargo testを走らせた時にだけ、 テストコードをコンパイルし走らせるよう指示します。これにより、ライブラリをビルドしたいだけの時にはコンパイルタイムを節約し、 テストが含まれないので、コンパイル後の成果物のサイズも節約します。結合テストは別のディレクトリに存在することになるので、 #[cfg(test)]注釈は必要ないとわかるでしょう。しかしながら、単体テストはコードと同じファイルに存在するので、 #[cfg(test)]を使用してコンパイル結果に含まれないよう指定するのです。

この章の最初の節で新しいadderプロジェクトを生成した時に、Cargoがこのコードも生成してくれたことを思い出してください:

ファイル名: src/lib.rs

```rust
#![allow(unused)]
fn main() {
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
}
```

このコードが自動生成されたテストモジュールです。cfgという属性は、configurationを表していて、 コンパイラに続く要素が、ある特定の設定オプションを与えられたら、含まれるように指示します。 今回の場合、設定オプションは、testであり、言語によって提供されているテストをコンパイルし、 走らせるためのものです。cfg属性を使用することで、cargo testで積極的にテストを実行した場合のみ、 Cargoがテストコードをコンパイルします。これには、このモジュールに含まれるかもしれないヘルパー関数全ても含まれ、 #[test]で注釈された関数だけにはなりません。

## 非公開関数をテストする

テストコミュニティ内で非公開関数を直接テストするべきかについては議論があり、 他の言語では非公開関数をテストするのは困難だったり、不可能だったりします。 あなたがどちらのテストイデオロギーを支持しているかに関わらず、Rustの公開性規則により、 非公開関数をテストすることが確かに可能です。リスト11-12の非公開関数internal_adderを含むコードを考えてください。

ファイル名: src/lib.rs

```rust
#![allow(unused)]
fn main() {
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
}
```

> ここではinternal_adderの関数が非公開です。他の言語ではこの非公開な関数についてテストしなくてもいいんじゃない？という動きがあると主張しているのでしょう。

リスト11-12: 非公開関数をテストする

internal_adder関数はpubとマークされていないものの、テストも単なるRustのコードであり、 testsモジュールもただのモジュールでしかないので、テスト内でinternal_adderを普通にインポートし呼び出すことができます。 非公開関数はテストするべきではないとお考えなら、Rustにはそれを強制するものは何もありません。

> これは考えによります。まずはテストがどういうものなのか把握する必要があるので、どんどんテストは書いていきましょう！

## 結合テスト
Rustにおいて、結合テストは完全にライブラリ外のものです。他のコードと全く同様にあなたのライブラリを使用するので、 ライブラリの公開APIの一部である関数しか呼び出すことはできません。その目的は、 ライブラリのいろんな部分が共同で正常に動作しているかをテストすることです。 単体では正常に動くコードも、結合した状態だと問題を孕む可能性もあるので、 結合したコードのテストの範囲も同様に重要になるのです。結合テストを作成するには、 まずtestsディレクトリが必要になります。

> 「結合テストを作成するには、まずtestsディレクトリが必要になります

## testsディレクトリ
プロジェクトディレクトリのトップ階層、srcの隣にtestsディレクトリを作成します。 Cargoは、このディレクトリに結合テストのファイルを探すことを把握しています。 そして、このディレクトリ内にいくらでもテストファイルを作成することができ、 Cargoはそれぞれのファイルを個別のクレートとしてコンパイルします。

結合テストを作成しましょう。リスト11-12のコードがsrc/lib.rsファイルにあるまま、 testsディレクトリを作成し、tests/integration_test.rsという名前の新しいファイルを生成し、 リスト11-13のコードを入力してください。

ファイル名: tests/integration_test.rs

``` rust
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

リスト11-13: adderクレートの関数の結合テスト

コードの頂点にextern crate adderを追記しましたが、これは単体テストでは必要なかったものです。 理由は、testsディレクトリのテストはそれぞれ個別のクレートであるため、 各々ライブラリをインポートする必要があるためです。

tests/integration_test.rsのどんなコードも#[cfg(test)]で注釈する必要はありません。 Cargoはtestsディレクトリを特別に扱い、cargo testを走らせた時にのみこのディレクトリのファイルをコンパイルするのです。 さあ、cargo testを実行してください:

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running target/debug/deps/adder-abcabcabc

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-ce99bcc2479f4607

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

3つの区域の出力が単体テスト、結合テスト、ドックテストを含んでいます。単体テスト用の最初の区域は、 今まで見てきたものと同じです: 各単体テストに1行(リスト11-12で追加したinternalという名前のもの)と、 単体テストのサマリー行です。

結合テストの区域は、 Running target/debug/deps/integration-test-ce99bcc2479f4607という行で始まっています(最後のハッシュはあなたの出力とは違うでしょう)。 次に、この結合テストの各テスト関数用の行があり、Doc-tests adder区域が始まる直前に、 結合テストの結果用のサマリー行があります。

単体テスト関数を追加することで単体テスト区域のテスト結果の行が増えたように、 作成した結合テストファイルにテスト関数を追加することでそのファイルの区域に結果の行が増えることになります。 結合テストファイルはそれぞれ独自の区域があるため、testsディレクトリにさらにファイルを追加すれば、 結合テストの区域が増えることになるでしょう。

それでも、テスト関数の名前を引数としてcargo testに指定することで、特定の結合テスト関数を走らせることができます。 特定の結合テストファイルにあるテストを全て走らせるには、cargo testに--test引数、 その後にファイル名を続けて使用してください:

> 1つの結合テストは1つのファイルで実施する

```
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

このコマンドは、tests/integration_test.rsファイルにあるテストのみを実行します。

## 結合テスト内のサブモジュール
結合テストを追加するにつれて、testsディレクトリに2つ以上のファイルを作成して体系化したくなるかもしれません; 例えば、テスト対象となる機能でテスト関数をグループ化することができます。前述したように、 testsディレクトリの各ファイルは、個別のクレートとしてコンパイルされます。

各結合テストファイルをそれ自身のクレートとして扱うと、 エンドユーザがあなたのクレートを使用するかのように個別のスコープを生成するのに役立ちます。 ですが、これはtestsディレクトリのファイルが、コードをモジュールとファイルに分ける方法に関して第7章で学んだように、 srcのファイルとは同じ振る舞いを共有しないことを意味します。

testsディレクトリのファイルの異なる振る舞いは、複数の結合テストファイルで役に立ちそうなヘルパー関数ができ、 第7章の「モジュールを別のファイルに移動する」節の手順に従って共通モジュールに抽出しようとした時に最も気付きやすくなります。 例えば、tests/common.rsを作成し、そこにsetupという名前の関数を配置したら、 複数のテストファイルの複数のテスト関数から呼び出したいsetupに何らかのコードを追加することができます:

ファイル名: tests/common.rs

```rust
#![allow(unused)]
fn main() {
pub fn setup() {
    // ここにライブラリテスト固有のコードが来る
    // setup code specific to your library's tests would go here
}
}
```

再度テストを実行すると、common.rsファイルは何もテスト関数を含んだり、setup関数をどこかから呼んだりしてないのに、 テスト出力にcommon.rs用の区域が見えるでしょう。

```
running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/common-b8b07b6f1be2db70 // これは望んだ結果ではない。

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-d993c68b431d39df

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

commonがrunning 0 testsとテスト結果に表示されるのは、望んだ結果ではありません。 ただ単に他の結合テストファイルと何らかのコードを共有したかっただけです。

commonがテスト出力に出現するのを防ぐには、tests/common.rsを作成する代わりに、 tests/common/mod.rsを作成します。第7章の「モジュールファイルシステムの規則」節において、 module_name/mod.rsという命名規則をサブモジュールのあるモジュールのファイルに使用しました。 ここではcommonにサブモジュールはありませんが、 このように命名することでコンパイラにcommonモジュールを結合テストファイルとして扱わないように指示します。 setup関数のコードをtests/common/mod.rsに移動し、tests/common.rsファイルを削除すると、 テスト出力に区域はもう表示されなくなります。testsディレクトリのサブディレクトリ内のファイルは個別クレートとしてコンパイルされたり、 テスト出力に区域が表示されることがないのです。

tests/common/mod.rsを作成した後、それをどの結合テストファイルからもモジュールとして使用することができます。 こちらは、tests/integration_test.rs内のit_adds_twoテストからsetup関数を呼び出す例です:

ファイル名: tests/integration_test.rs

``` rust
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

mod common;という宣言は、リスト7-21で模擬したモジュール宣言と同じであることに注意してください。それから、テスト関数内でcommon::setup()関数を呼び出すことができます。

> testsの配下だと結合テストファイルとして認識されているのかな？またサブディレクトリ以下のファイルは結合テストファイルとして認識されるのかな？

## バイナリクレート用の結合テスト
もしもプロジェクトがsrc/main.rsファイルのみを含み、src/lib.rsファイルを持たないバイナリクレートだったら、 testsディレクトリに結合テストを作成し、 extern crateを使用してsrc/main.rsファイルに定義された関数をインポートすることはできません。 ライブラリクレートのみが、他のクレートが呼び出して使用できる関数を晒せるのです; バイナリクレートはそれ単体で実行することを意味しています。

これは、バイナリを提供するRustのプロジェクトに、 src/lib.rsファイルに存在するロジックを呼び出す単純なsrc/main.rsファイルがある一因になっています。 この構造を使用して結合テストは、extern crateを使用して重要な機能を用いることでライブラリクレートをテストすることができます。 この重要な機能が動作すれば、src/main.rsファイルの少量のコードも動作し、その少量のコードはテストする必要がないわけです。

# まとめ
Rustのテスト機能は、変更を加えた後でさえ想定通りにコードが機能し続けることを保証して、 コードが機能すべき方法を指定する手段を提供します。単体テストはライブラリの異なる部分を個別に用い、 非公開の実装詳細をテストすることができます。結合テストは、ライブラリのいろんな部分が共同で正常に動作することを確認し、 ライブラリの公開APIを使用して外部コードが使用するのと同じ方法でコードをテストします。 Rustの型システムと所有権ルールにより防がれるバグの種類もあるものの、それでもテストは、 コードが振る舞うと予想される方法に関するロジックのバグを減らすのに重要なのです。

この章と以前の章で学んだ知識を結集して、とあるプロジェクトに取り掛かりましょう！