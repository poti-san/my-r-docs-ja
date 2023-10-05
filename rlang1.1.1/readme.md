---
title: rlang 1.1.1 readme
---

{%raw%}

**訳者註：関数等のリンクはほとんどリンク切れです。和訳でき次第追加します。**

# ![](figures/logo.png) rlang 1.1.1

rlangはRプログラミングで使うフレームワークとAPIのコレクションです。

## フレームワーク

rlangは2つの包括的なフレームワークを定義します。

- **tidy評価**：dplyrやggplot2のようなtidyverseパッケージで使うプログラム可能な[データマスク](topic-data-mask.html)フレームワークです。ユーザーとしてはエンブレース演算子[`{{`](reference/embrace-operator.html)や[glueパッケージ](https://glue.tidyverse.org/)の演算子[`"{"`と`"{{"`](reference/glue-operators.html)を用いた名前注入で出会うでしょう。
- **rlangエラーズ**：エラー通知・表示のツール集です。[`global_entrace()`](reference/global_entrace.html)によるバックトレースのキャプチャや[`last_error()`](reference/last_error.html)と[`last_warnings()`](reference/last_warnings.html)によるバックトレースの表示を含みます。[`abort()`](reference/abort.html)を使えば箇条書き、構造化メタデータ、エラー連鎖サポートを使ってエラーを作成できます。エラーメッセージの表示は箇条書きと連鎖エラーに最適化されており、オプションでcliパッケージ（[`local_use_cli()`](reference/local_use_cli.html)を参照）と統合できます。

## 引数の摂取

引数のチェック、検証、前処理を簡単にするツール集です。

- 関数引数のチェック：[`arg_match()`](reference/arg_match.html)、[`check_required()`](reference/check_required.html)、[`check_exclusive()`](reference/check_exclusive.html)等。
- ドットのチェック：[`check_dots_used()`](reference/check_dots_used.html)、[`check_dots_empty()`](reference/check_dots_empty.html)等。
- [動的ドット](reference/dyn-dots.html)の収集：[`list2()`](reference/list2.html)等。動的ドットは[`!!!`](reference/splice-operator.html)によるスプライシングや[glueパッケージ](https://glue.tidyverse.org/)演算子[`"{"`](reference/glue-operators.html)、[`"{{"`](reference/glue-operators.html)による名前注入に対応しています。

## プログラミングインターフェイス

rlangはRやRオブジェクトと使える多様なインターフェイスを提供します。

- Rセッション：[`check_installed()`](reference/is_installed.html)、[`on_load()`](reference/on_load.html)、[`on_package_load()`](reference/on_load.html)等。
- 環境：[`env()`](reference/env.html)、[`env_has()`](reference/env_has.html)、[`env_get()`](reference/env_get.html)、[`env_bind()`](reference/env_bind.html)、[`env_unbind()`](reference/env_unbind.html)、[`env_print()`](reference/env_print.html)、[`local_bindings()`](reference/local_bindings.html)等。
- 評価：[`inject()`](reference/inject.html)、[`eval_bare()`](reference/eval_bare.html)等。
- 呼び出しとシンボル：[`call2()`](reference/call2.html)、[`is_call()`](reference/is_call.html)、[`is_call_simple()`](reference/call_name.html)、[`data_sym()`](reference/sym.html)、[`data_syms()`](reference/sym.html)等。
- 関数：[`new_function()`](reference/new_function.html)、[`as_function()`](reference/as_function.html)等。後者はラムダ関数でpurrr形式の式表記に対応しています。

## インストール

公開版はCRANからインストールできます。

```
install.packages("rlang")
```

開発版はGitHubからインストールできます。

```
# install.packages("pak")
pak::pkg_install("r-lib/rlang")
```

## 行動規範

rlangプロジェクトは[寄与者の行動規範（原文）](https://rlang.r-lib.org/CODE_OF_CONDUCT.htmlml)を伴って公開されることに注意してください。このプロジェクトに寄与するには行動規範を遵守する必要があります。

{%endraw%}