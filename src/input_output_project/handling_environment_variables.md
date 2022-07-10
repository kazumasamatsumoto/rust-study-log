# 環境変数を取り扱う
おまけの機能を追加してminigrepを改善します: 環境変数でユーザがオンにできる大文字小文字無視の検索用のオプションです。 この機能をコマンドラインオプションにして、適用したい度にユーザが入力しなければならないようにすることもできますが、 代わりに環境変数を使用します。そうすることでユーザは1回環境変数をセットすれば、そのターミナルセッションの間は、 大文字小文字無視の検索を行うことができるようになるわけです。

> 大文字小文字の区別です。（これ結構楽しい）

## 大文字小文字を区別しないsearch関数用に失敗するテストを書く
環境変数がオンの場合に呼び出すsearch_case_insensitive関数を新しく追加したいです。テスト駆動開発の過程に従い続けるので、 最初の手順は、今回も失敗するテストを書くことです。新しいsearch_case_insensitive関数用の新規テストを追加し、 古いテストをone_resultからcase_sensitiveに名前変更して、二つのテストの差異を明確化します。 リスト12-20に示したようにですね。

ファイル名: src/lib.rs

``` rust

#![allow(unused)]
fn main() {
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
// Rust
// 安全かつ高速で生産的
// 三つを選んで
// ガムテープ
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
// (最後の行のみ)
// 私を信じて
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
}
```

リスト12-20: 追加しようとしている大文字小文字を区別しない関数用の失敗するテストを新しく追加する

古いテストのcontentsも変更していることに注意してください。大文字小文字を区別する検索を行う際に、 "duct"というクエリに合致しないはずの大文字Dを使用した"Duct tape"(ガムテープ)という新しい行を追加しました。 このように古いテストを変更することで、既に実装済みの大文字小文字を区別する検索機能を誤って壊してしまわないことを保証する助けになります。 このテストはもう通り、大文字小文字を区別しない検索に取り掛かっても通り続けるはずです。

大文字小文字を区別しない検索の新しいテストは、クエリに"rUsT"を使用しています。 追加直前のsearch_case_insensitive関数では、"rUsT"というクエリは、 両方ともクエリとは大文字小文字が異なるのに、大文字Rの"Rust:"を含む行と、 “Trust me.”という行にもマッチするはずです。これが失敗するテストであり、まだsearch_case_insensitive関数を定義していないので、 コンパイルは失敗するでしょう。リスト12-16のsearch関数で行ったのと同様に空のベクタを常に返すような仮実装を追加し、テストがコンパイルされるものの、失敗する様をご自由に確認してください。

## search_case_insensitive関数を実装する
search_case_insensitive関数は、リスト12-21に示しましたが、search関数とほぼ同じです。 唯一の違いは、queryと各lineを小文字化していることなので、入力引数の大文字小文字によらず、 行がクエリを含んでいるか確認する際には、同じになるわけです。

ファイル名: src/lib.rs
```rust

#![allow(unused)]
fn main() {
pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
}
```

リスト12-21: 比較する前にクエリと行を小文字化するよう、search_case_insensitive関数を定義する

まず、query文字列を小文字化し、同じ名前の覆い隠された変数に保存します。ユーザのクエリが"rust"や"RUST"、 "Rust"、"rUsT"などだったりしても、"rust"であり、大文字小文字を区別しないかのようにクエリを扱えるように、 to_lowercaseをクエリに対して呼び出すことは必須です。

queryは最早、文字列スライスではなくStringであることに注意してください。というのも、 to_lowercaseを呼び出すと、既存のデータを参照するというよりも、新しいデータを作成するからです。 例として、クエリは"rUsT"だとしましょう: その文字列スライスは、小文字のuやtを使えるように含んでいないので、 "rust"を含む新しいStringのメモリを確保しなければならないのです。今、containsメソッドに引数としてqueryを渡すと、 アンド記号を追加する必要があります。containsのシグニチャは、文字列スライスを取るよう定義されているからです。

次に、各lineがqueryを含むか確かめる前にto_lowercaseの呼び出しを追加し、全文字を小文字化しています。 今やlineとqueryを小文字に変換したので、クエリが大文字であろうと小文字であろうとマッチを検索するでしょう。

この実装がテストを通過するか確認しましょう:

```
running 2 tests
test test::case_insensitive ... ok
test test::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

素晴らしい！どちらも通りました。では、run関数から新しいsearch_case_insensitive関数を呼び出しましょう。 1番目に大文字小文字の区別を切り替えられるよう、Config構造体に設定オプションを追加します。 まだどこでも、このフィールドの初期化をしていないので、追加するとコンパイルエラーが起きます:

ファイル名: src/lib.rs

``` rust

#![allow(unused)]
fn main() {
pub struct Config {
    pub query: String,
    pub filename: String,
    pub case_sensitive: bool,
}
}
```

論理値を持つcase_sensitiveフィールドを追加したことに注意してください。次に、run関数に、 case_sensitiveフィールドの値を確認し、search関数かsearch_case_insensitive関数を呼ぶかを決定するのに使ってもらう必要があります。 リスト12-22のようにですね。それでも、これはまだコンパイルできないことに注意してください。

ファイル名: src/lib.rs

``` rust
#![allow(unused)]
#use std::error::Error;
#use std::fs;
#use std::env;
#
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.filename)?;

    let results = if config.case_sensitive {
      search(&config.query, &contents)
    } else {
      search_case_insensitive(&config.query, &contents)
    };

    for line in results {
        println!("{}", line);
    }

    Ok(())
}
#
#pub struct Config {
#    pub query: String,
#    pub filename: String,
#    pub case_sensitive: bool,
#}
#
#impl Config {
#    pub fn new(args: &[String]) -> Result<Config, &str> {
#        if args.len() < 3 {
#            return Err("not enough arguments");
#        }
#        let query = args[1].clone();
#        let filename = args[2].clone();
#
#        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();
#
#        Ok(Config { query, filename, case_sensitive })
#    }
#}
#
#pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#    let mut results = Vec::new();
#
#    for line in contents.lines() {
#        if line.contains(query) {
#            results.push(line);
#        }
#    }
#
#    results
#}
#
#pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#    let query = query.to_lowercase();
#    let mut results = Vec::new();
#
#    for line in contents.lines() {
#      if line.to_lowercase().contains(&query) {
#        results.push(line);
#      }
#    }
#
#    results
#}
#
# #[cfg(test)]
#mod tests {
#    use super::*;
#
#    #[test]
#    fn case_sensitive() {
#        let query = "duct";
#        let contents = "\
#Rust:
#safe, fast, productive.
#Pick three.
#Duct tape";
#        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
#    }
#
#    #[test]
#    fn case_insensitive() {
#        let query = "rUsT";
#        let contents = "\
#Rust:
#safe, fast, productive.
#Pick three.
#Trust me.";
#        assert_eq!(
#            vec!["Rust:", "Trust me."],
#            search_case_insensitive(query, contents)
#        )
#    }
#}
#
#
```

リスト12-22: config.case_sensitiveの値に基づいてsearchかsearch_case_insensitiveを呼び出す

最後に、環境変数を確認する必要があります。環境変数を扱う関数は、標準ライブラリのenvモジュールにあるので、 use std::env;行でsrc/lib.rsの冒頭でそのモジュールをスコープに持ってくる必要があります。そして、 envモジュールからvar関数を使用してCASE_INSENSITIVEという環境変数のチェックを行います。 リスト12-23のようにですね。

ファイル名: src/lib.rs

```rust
use std::env;

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }
        let query = args[1].clone();
        let filename = args[2].clone();

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

リスト12-23: CASE_INSENSITIVEという環境変数のチェックを行う

ここで、case_sensitiveという新しい変数を生成しています。その値をセットするために、 env::var関数を呼び出し、CASE_INSENSITIVE環境変数の名前を渡しています。env::var関数は、 環境変数がセットされていたら、環境変数の値を含むOk列挙子の成功値になるResultを返します。 環境変数がセットされていなければ、Err列挙子を返すでしょう。

Resultのis_errメソッドを使用して、エラーでありゆえに、セットされていないことを確認しています。 これは大文字小文字を区別する検索をすべきことを意味します。CASE_INSENSITIVE環境変数が何かにセットされていれば、 is_errはfalseを返し、プログラムは大文字小文字を区別しない検索を実行するでしょう。環境変数の値はどうでもよく、 セットされているかどうかだけ気にするので、unwrapやexpectあるいは、他のここまで見かけたResultのメソッドではなく、 is_errをチェックしています。

case_sensitive変数の値をConfigインスタンスに渡しているので、リスト12-22で実装したように、 run関数はその値を読み取り、searchかsearch_case_insensitiveを呼び出すか決定できるのです。

試行してみましょう！まず、環境変数をセットせずにクエリはtoでプログラムを実行し、 この時は全て小文字で"to"という言葉を含むあらゆる行が合致するはずです。

```
$ cargo run to poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
```

まだ機能しているようです！では、CASE_INSENSITIVEを1にしつつ、同じクエリのtoでプログラムを実行しましょう。

PowerShellを使用しているなら、1コマンドではなく、2コマンドで環境変数をセットし、プログラムを実行する必要があるでしょう:

```
$ export CASE_INSENSITIVE=true
$ cargo run to poem.txt
```

大文字も含む可能性のある"to"を含有する行が得られるはずです:

解除する場合は

```
$ unset CASE_INSENSITIVE
```

> ここはいい感じに楽しかった。