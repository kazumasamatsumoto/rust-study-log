# クロージャ: 環境をキャプチャできる匿名関数

Rust のクロージャは、変数に保存したり、引数として他の関数に渡すことのできる匿名関数です。 ある場所でクロージャを生成し、それから別の文脈でクロージャを呼び出して評価することができます。 関数と異なり、呼び出されたスコープの値をクロージャは、キャプチャすることができます。 これらのクロージャの機能がコードの再利用や、動作のカスタマイズを行わせてくれる方法を模擬しましょう。

> この時点ではちょっと何いっているかわからない。

## クロージャで動作の抽象化を行う

クロージャを保存して後々使用できるようにするのが有用な場面の例に取り掛かりましょう。その過程で、 クロージャの記法、型推論、トレイトについて語ります。

以下のような架空の場面を考えてください: カスタマイズされたエクササイズのトレーニングプランを生成するアプリを作るスタートアップで働くことになりました。 バックエンドは Rust で記述され、トレーニングプランを生成するアルゴリズムは、アプリユーザの年齢や、 BMI、運動の好み、最近のトレーニング、指定された強弱値などの多くの要因を考慮します。 実際に使用されるアルゴリズムは、この例では重要ではありません; 重要なのは、この計算が数秒要することです。 必要なときだけこのアルゴリズムを呼び出し、1 回だけ呼び出したいので、必要以上にユーザを待たせないことになります。

> トレーニングアプリ
>
> 目的：
>
> 必要な時だけアルゴリズムを呼び出し、一回だけ呼び出したい。必要以上にユーザを待たせないこと。

リスト 13-1 に示した simulated_expensive_calculation 関数でこの仮定のアルゴリズムを呼び出すことをシミュレートし、 この関数は calculating slowly と出力し、2 秒待ってから、渡した数値をなんでも返します。

ファイル名: src/main.rs

```rust

#![allow(unused)]
fn main() {
use std::thread;
use std::time::Duration;

fn simulated_expensive_calculation(intensity: u32) -> u32 {
    // ゆっくり計算します
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
}
```

リスト13-1: 実行に約2秒かかる架空の計算の代役を務める関数

次は、この例で重要なトレーニングアプリの部分を含むmain関数です。この関数は、 ユーザがトレーニングプランを要求した時にアプリが呼び出すコードを表します。 アプリのフロントエンドと相互作用する部分は、クロージャの使用と関係ないので、プログラムへの入力を表す値をハードコードし、 その出力を表示します。

必要な入力は以下の通りです:

- ユーザの強弱値、これはユーザがトレーニングを要求して、低強度のトレーニングか、 高強度のトレーニングがしたいかを示したときに指定されます。
- 乱数、これはトレーニングプランにバリエーションを起こします。

出力は、推奨されるトレーニングプランになります。リスト13-2は使用するmain関数を示しています。

ファイル名: src/main.rs

```rust
fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(
        simulated_user_specified_value,
        simulated_random_number
    );
}
fn generate_workout(intensity: u32, random_number: u32) {}
```

リスト13-2: ユーザ入力や乱数生成をシミュレートするハードコードされた値があるmain関数

簡潔性のために、変数simulated_user_specified_valueは10、変数simulated_random_numberは7とハードコードしました; 実際のプログラムにおいては、強弱値はアプリのフロントエンドから取得し、乱数の生成には、第2章の数当てゲームの例のように、randクレートを使用するでしょう。 main関数は、シミュレートされた入力値とともにgenerate_workout関数を呼び出します。

今や文脈ができたので、アルゴリズムに取り掛かりましょう。リスト13-3のgenerate_workout関数は、 この例で最も気にかかるアプリのビジネスロジックを含んでいます。この例での残りの変更は、 この関数に対して行われるでしょう:

ファイル名: src/main.rs

```rust
fn generate_workout(intensity: u32, random_number: u32) {
    if intensity < 25 {

        println!(
            // 今日は{}回腕立て伏せをしてください！
            "Today, do {} pushups!",
            simulated_expensive_calculation(intensity)
        );

        println!(
            // 次に、{}回腹筋をしてください！
            "Next, do {} situps!",
            simulated_expensive_calculation(intensity)
        );
    } else {
        if random_number == 3 {
            // 今日は休憩してください！水分補給を忘れずに！
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                // 今日は、{}分間走ってください！
                "Today, run for {} minutes!",
                simulated_expensive_calculation(intensity)
            );
        }
    }
}
```

リスト13-3: 入力に基づいてトレーニングプランを出力するビジネスロジックと、 simulated_expensive_calculation関数の呼び出し

リスト13-3のコードには、遅い計算を行う関数への呼び出しが複数あります。最初のifブロックが、 simulated_expensive_calculationを2回呼び出し、外側のelse内のifは全く呼び出さず、 2番目のelseケースの内側にあるコードは1回呼び出しています。

generate_workout関数の期待される振る舞いは、まずユーザが低強度のトレーニング(25より小さい数値で表される)か、 高強度のトレーニング(25以上の数値)を欲しているか確認することです。

低強度のトレーニングプランは、シミュレーションしている複雑なアルゴリズムに基づいて、 多くの腕立て伏せや腹筋運動を推奨してきます。

ユーザが高強度のトレーニングを欲していれば、追加のロジックがあります: アプリが生成した乱数がたまたま3なら、 アプリは休憩と水分補給を勧めます。そうでなければ、ユーザは複雑なアルゴリズムに基づいて数分間のランニングをします。

このコードは現在、ビジネスのほしいままに動くでしょうが、データサイエンスチームが、 simulated_expensive_calculation関数を呼び出す方法に何らかの変更を加える必要があると決定したとしましょう。 そのような変更が起きた時に更新を簡略化するため、simulated_expensive_calculation関数を1回だけ呼び出すように、 このコードをリファクタリングしたいです。また、その過程でその関数への呼び出しを増やすことなく無駄に2回、 この関数を現時点で呼んでいるところを切り捨てたくもあります。要するに、結果が必要なければ関数を呼び出したくなく、 それでも1回だけ呼び出したいのです。

## 関数でリファクタリング
多くの方法でトレーニングプログラムを再構築することもできます。 1番目にsimulated_expensive_calculation関数への重複した呼び出しを変数に抽出しようとしましょう。リスト13-4に示したように。

ファイル名: src/main.rs

```rust
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_result =
        simulated_expensive_calculation(intensity);

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result
        );
        println!(
            "Next, do {} situps!",
            expensive_result
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result
            );
        }
    }
}
```

リスト13-4: 複数のsimulated_expensive_calculationの呼び出しを1箇所に抽出し、 結果をexpensive_result変数に保存する

この変更によりsimulated_expensive_calculationの呼び出しが単一化され、 最初のifブロックが無駄に関数を2回呼んでいた問題を解決します。不幸なことに、これでは、 あらゆる場合にこの関数を呼び出し、その結果を待つことになり、結果値を全く使用しない内側のifブロックでもそうしてしまいます。

プログラムの1箇所でコードを定義したいですが、結果が本当に必要なところでだけコードを実行します。 これは、クロージャのユースケースです！
