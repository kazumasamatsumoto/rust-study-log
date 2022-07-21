# 1日目

まずは登録から始めました。

面談も実施して、基本的に優しい感じの人でした。

次に、カード決済を実施しました。8万円ぐらいかかったので、会社に相談しました。

とても高かったです。

登録が完了したらパスワードの変更を実施しました。

その次にDiscordのチャンネルへ移動して、自己紹介をしました。

今日から色々勉強していこうと思います！頑張るぞ！

大体1日1時間ぐらい勉強する感じで継続していこう。

##　環境構築

・rustの環境構築です

[環境構築のページ](https://rustup.rs/)

・ターミナルで以下を実行し、画面の指示に従ってください。

``` zsh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

・エディタはVSCodeを使用します。

[VSCode](https://code.visualstudio.com/)

## VSCodeのプラグイン

### Rust Analyzer

[rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer)

- Visual Studio CodeのRust言語サポート
- この拡張モジュールは、プログラミング言語 Rust のサポートを提供します。rust-lang.rustを置き換えるものとして推奨されます。

#### 機能

- インポートの挿入によるコード補完
- 定義、実装、型定義への移動
- 全参照の検索、ワークスペースのシンボル検索、シンボル名の変更
- ホバーでの型とドキュメントの表示
- 型とパラメータ名のインレイヒント
- セマンティックシンタックスハイライト
- 豊富なアシスト機能（コードアクション）
- エラーからの提案の適用
- ... その他多数、マニュアルでご確認ください。

#### クイックスタート
- rustupをインストールします。
- rust-analyzerエクステンションをインストールします。
- コンフィギュレーション
- このエクステンションは、VSCodeのコンフィギュレーション設定を通じて設定を提供します。すべてのコンフィギュレーションはrust-analyzer.*の下にあります。

VSCode固有のコンフィギュレーションの詳細についてはマニュアルを参照してください。

#### コミュニケーション
使い方やトラブルシューティングの依頼は、Rustフォーラムの「IDEs and Editors」カテゴリをご利用ください。

#### ドキュメンテーション
詳細はrust-analyzer.github.ioを参照してください。

### CodeLLDB

[CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)

#### 概要
LLDBを搭載したネイティブデバッガ。C++、Rust、その他のコンパイル済み言語をデバッグできます。

#### 機能紹介
条件付きブレークポイント、関数ブレークポイント、ログポイント
ハードウェアデータアクセスブレークポイント(ウォッチポイント)
統合ターミナルまたは外部ターミナルでのデバッガ起動。
命令レベルのステップ実行による逆アセンブルビュー。
ロードされたモジュールの表示。
Pythonスクリプト。
高度な視覚化のためのHTMLレンダリング
起動設定のためのワークスペースレベルのデフォルト。
リモートデバッグ。
リバースデバッグ(実験的、互換性のあるバックエンドが必要)
詳しくはユーザーズマニュアルをご覧ください。

#### 言語
CodeLLDB には、ベクトル、文字列、マップ、その他の標準ライブラリ型のビジュアライザーが組み込まれています。
また、Ada、Fortran、Kotlin Native、Nim、Objective-C、Pascal、Swift、Zigなど、コンパイラが互換性のあるデバッグ情報を生成する他のほとんどのコンパイル済み言語でも使用可能です。

#### 対応プラットフォーム
ホスト
glibc 2.18+ が動作する Linux (x86_64、aarch64、armhf 用)。
MacOS X 10.10+（x86_64）、11.0+（arm64）。
Windows 10 (x86_64)。
対象
CodeLLDBはAArch64, ARM, AVR, MSP430, RISCV, X86アーキテクチャをサポートし、リモートデバッグにより組み込みプラットフォームでのデバッグに使用することができます。

#### より詳細な情報
CodeLLDB ユーザーズマニュアル - この拡張機能の使用方法です。
VS Code でのデバッグ - VSCode デバッグを初めて使用する場合。
LLDB チュートリアル - LLDB の CLI コマンドとスクリプト機能のすべてを CodeLLDB で使用することができます。
Wikiページ - トラブルシューティングやその他のTipsやTricks。
ディスカッション - 質問やディスカッションを行うことができます。
スクリーンショット
データ可視化によるC++デバッグ (Howto)。
ソース

Rustデバッギング。
ソース

### Better TOML

[Better TOML](https://marketplace.visualstudio.com/items?itemName=bungcip.better-toml)

better-toml README
Better TOML は、TOML ファイルをサポートするための対コード拡張です。

### VS Codeのオプションプラグインです。

Error Lens - インラインエラーを取得します。
crates - Cargo.tomlにある古い依存関係をハイライトします。
Tabnine AI - AIコードの補完