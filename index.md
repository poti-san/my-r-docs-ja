---
title: R関係の私的な和訳
---

{% raw %}
## あばうと

- 趣味のプログラマーによるR言語関係の翻訳集です。GitHubも翻訳も手探りですが、誰かの助けになったら幸いです。
- ちょこちょこ修正する予定です。おかしな点があればお気軽にご指摘ください。

## rlang 1.1.1 readme

[readme.md](rlang1.1.1/readme.md)

## rlang 1.1.1 トピックス

- tidyverseのtidy評価機能を提供する[rlangパッケージ](https://rlang.r-lib.org/index.html)バージョン1.1.1のトピック和訳です。
- rlang 1.1.1はrlang authorsの著作物でMITライセンスです。詳細は[rlangのフルライセンス表記](https://rlang.r-lib.org/LICENSE.html)を参照してください。以下の翻訳もMITライセンスに従います。なお、`hash()`はBSD 2-Clause LicenseのxxHashライブラリーを使用しています。詳細は[rlangのライセンス表記](https://github.com/r-lib/rlang/blob/main/LICENSE.note)を参照してください。
- 原文は最新版のため、和訳と内容が異なる可能性があります。その場合はrlang GitHubリポジトリから対応バージョンのZIPファイルをダウンロードしてmanディレクトリ内の同名ファイルを確認してください。

### tidy評価
- [概観] [データマスク処理とは何でなぜ`{{`が必要なのか？](rlang1.1.1/man/topic-data-mask.md)／[原文](https://rlang.r-lib.org/reference/topic-data-mask.html "What is data-masking and why do I need `{{`?")
- [概観] [データマスクプログラミングパターン](rlang1.1.1/man/topic-data-mask-programming.md)／[原文](https://rlang.r-lib.org/reference/topic-data-mask-programming.html "Data mask programming patterns")
- [ガイド] [データマスクの曖昧さ](rlang1.1.1/man/topic-data-mask-ambiguity.md)／[原文](https://rlang.r-lib.org/reference/topic-data-mask-ambiguity.html "The data mas ambiguity")
- [ガイド] [二重評価問題](rlang1.1.1/man/topic-double-evaluation.md)／[原文](https://rlang.r-lib.org/reference/topic-double-evaluation.html "The double evaluation problem")
- [ノート] [コンテキスト外で注入演算子を使うと何が起こるか？](rlang1.1.1/man/topic-inject-out-of-context.md)／[原文](https://rlang.r-lib.org/reference/topic-inject-out-of-context.html "What happens if I use injection operators out of context?")
- [ノート] [`{{`は通常のオブジェクトとして機能するか？](rlang1.1.1/man/topic-embrace-non-args.md)／[原文](https://rlang.r-lib.org/reference/topic-embrace-non-args.html "Does `{{` work on regular objects?")
### メタプログラミング
- [概観] [R式のディフューズ](rlang1.1.1/man/topic-defuse.md)／[原文](https://rlang.r-lib.org/reference/topic-defuse.html "Defusing R expressions")
- [概観] [`!!`、`!!!`、glue構文による注入](rlang1.1.1/man/topic-inject.md)／[原文](https://rlang.r-lib.org/reference/topic-inject.html "Injecting with `!!`, `!!!`, and glue syntax")
- [概観] [メタプログラミングパターン](rlang1.1.1/man/topic-metaprogramming.md)／[原文](https://rlang.r-lib.org/reference/topic-metaprogramming.html "Metaprogramming patterns")
- [概観] [quosureとは何でいつ必要か？](rlang1.1.1/man/topic-quosure.md)／[原文](https://rlang.r-lib.org/reference/topic-quosure.html "What are quosures and when are they needed?")
- [ガイド] [`...`を使わずに複数列を受け取る](rlang1.1.1/man/topic-multiple-columns.md)／[原文](https://rlang.r-lib.org/reference/topic-multiple-columns.html "Taking multiple columns without `...`")
- [ノート] [なぜ文字列やその他の定数は空の環境でquoseされるのか？](rlang1.1.1/man/topic-embrace-constants.md)／[原文](https://rlang.r-lib.org/reference/topic-embrace-constants.html "Why are strings and other constants enquosed in the empty environment?")
### 条件
- [ガイド] [エラーメッセージに関数呼び出しを含める](rlang1.1.1/man/topic-error-call.md)／[原文](https://rlang.r-lib.org/reference/topic-error-call.html "Including function calls in error messages")
- [ガイド] [エラー連鎖にコンテキスト情報を含める](rlang1.1.1/man/topic-error-chaining.md)／[原文](https://rlang.r-lib.org/reference/topic-error-chaining.html "Including contextual information with error chains")
- [ガイド] [cliを用いたメッセージの書式化](rlang1.1.1/man/topic-condition-formatting.md)／[原文](https://rlang.r-lib.org/reference/topic-condition-formatting.html "Formatting messages with cli")
- [ノート] [状態メッセージのカスタマイズ](rlang1.1.1/man/topic-condition-customisation.md)／[原文](https://rlang.r-lib.org/reference/topic-condition-customisation.html "Customising condition messages")

## GitHubリポジトリ

このGitHub PagesのGitHubリポジトリに関する備考です。

- ほとんどのマークダウンファイルでは全体または先頭のYAML以降をLiquidの`raw`&`endraw`タグで囲っています。GitHub Pagesが内部で使用するLiquidとtidyverseの`{{`が衝突して発生するエラーを回避するためです。Jekyllがバージョン4になったらYAMLのLiquid無効化設定と入れ替えます。

{% endraw %}
