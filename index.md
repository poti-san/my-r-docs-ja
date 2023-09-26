---
title: R関係の私的な和訳
---

{% raw %}
## あばうと

- 趣味のプログラマーによるR言語関係の翻訳集です。GitHubも翻訳も手探りですが、誰かの助けになったら幸いです。
- ちょこちょこ修正する予定です。おかしな点があればお気軽にご指摘ください。

## rlang 1.1.1 トピックス

- tidyverseのtidy評価機能を提供する[rlangパッケージ バージョン1.1.1](https://rlang.r-lib.org/index.html)のトピックス和訳です。
- rlangはrlang authorsの著作物でMITライセンスです。詳細は[rlangのフルライセンス表記](https://rlang.r-lib.org/LICENSE.html)を参照してください。以下の翻訳もMITライセンスに従います。

### tidy評価
- [概観] [データマスク処理とは何でなぜ`{{`が必要なのか？](rlang1.1.1-topics/topic-data-mask.md)　原文：[What is data-masking and why do I need `{{`?](https://rlang.r-lib.org/reference/topic-data-mask.html)
- [概観] [データマスクプログラミングパターン](rlang1.1.1-topics/topic-data-mask-programming.md)　原文：[Data mask programming patterns](https://rlang.r-lib.org/reference/topic-data-mask-programming.html)
- [ガイド] [データマスクの曖昧さ](rlang1.1.1-topics/topic-data-mask-ambiguity.md)　原文：[The data mask ambiguity](https://rlang.r-lib.org/reference/topic-data-mask-ambiguity.html)
- [ガイド] [二重評価問題](rlang1.1.1-topics/topic-double-evaluation.md)　原文：[The double evaluation problem](https://rlang.r-lib.org/reference/topic-double-evaluation.html)
- [ノート] 未翻訳　原文：[What happens if I use injection operators out of context?](https://rlang.r-lib.org/reference/topic-inject-out-of-context.html)
- [ノート] [`{{`は通常のオブジェクトとして機能するか？](rlang1.1.1-topics/topic-embrace-non-args.md)　原文：[Does `{{` work on regular objects?](https://rlang.r-lib.org/reference/topic-embrace-non-args.html)
### メタプログラミング
- [概観] [R式のディフューズ](rlang1.1.1-topics/topic-defuse.md)　原文：[Defusing R expressions](https://rlang.r-lib.org/reference/topic-defuse.html)
- [概観] [`!!`、`!!!`、glue構文による注入](rlang1.1.1-topics/topic-inject.md)　原文：[Injecting with `!!`, `!!!`, and glue syntax](https://rlang.r-lib.org/reference/topic-inject.html)
- [概観] 未翻訳　原文：[Metaprogramming patterns](https://rlang.r-lib.org/reference/topic-metaprogramming.html)
- [概観] 未翻訳　原文：[What are quosures and when are they needed?](https://rlang.r-lib.org/reference/topic-quosure.html)
- [ガイド] 未翻訳　原文：[Taking multiple columns without `...`](https://rlang.r-lib.org/reference/topic-multiple-columns.html)
- [ノート] [なぜ文字列やその他の定数は空の環境でquoseされるのか？](rlang1.1.1-topics/topic-embrace-constants.md)　原文：[Why are strings and other constants enquosed in the empty environment?](https://rlang.r-lib.org/reference/topic-embrace-constants.html)
### 条件
- [ガイド] [エラーメッセージに関数呼び出しを含める](rlang1.1.1-topics/topic-error-call.md)　原文：[Including function calls in error messages](https://rlang.r-lib.org/reference/topic-error-call.html)
- [ガイド] 未翻訳　原文：[Including contextual information with error chains](https://rlang.r-lib.org/reference/topic-error-chaining.html)
- [ガイド] [cliを用いたメッセージの書式化](rlang1.1.1-topics/topic-condition-formatting.md)　原文：[Formatting messages with cli](https://rlang.r-lib.org/reference/topic-condition-formatting.html)
- [ノート] [状態メッセージのカスタマイズ](rlang1.1.1-topics/topic-condition-customisation.md)　原文：[Customising condition messages](https://rlang.r-lib.org/reference/topic-condition-customisation.html)

## GitHubリポジトリ

このGitHub PagesのGitHubリポジトリに関する備考です。

- ほとんどのマークダウンファイルでは全体または先頭のYAML以降を`{% raw %}`～`{% endraw %}`で囲っています。GitHub Pagesが内部で使用するLiquidとtidyverseの`{{`が衝突して発生するエラーを回避するためです。Jekyllがバージョン4になったらYAMLのLiquid無効化設定と入れ替えます。

{% endraw %}
