---
parent: rlang 1.1.1 トピックス
title: cliを用いたメッセージの書式化
---
{% raw %}
# cliを用いたメッセージの書式化

状態の書式化はエラーメッセージの生入力に適用できる処理の一式です。次を含みます。

- 文字列ベクトル単位の行から指定幅で折り返されたエラー箇条書きへの変換。箇条書きで各項目ひとつずつ要点を伝えてメッセージの記述を簡単にします。
  
  ```r
  abort(c(
    "エラーヘッダー",
    "*" = "エラーバレット",
    "i" = "情報バレット",
    "x" = "クロスバレット"
  ))
  #> Error:
  #> ! エラーヘッダー
  #> * エラーバレット
  #> i 情報バレット
  #> x クロスバレット
  ```
  
エラーメッセージスタイルの詳細は[tidyverse error style guide](https://style.tidyverse.org/error-messages.html)を参照してください。

- スタイルの適用（強調、太字等）とメッセージ要素の色付け。

rlangパッケージには基本的な書式化ルーチンが組み込まれていますが、主な書式化エンジンは[cliパッケージ](https://cli.r-lib.org/)で実装されています。

### cliによるメッセージの書式化

既定ではrlangは箇条書きの書式化に内部機構を使用します。しかし、より望ましい書式化は[`cli::cli_abort()`](https://cli.r-lib.org/reference/cli_abort.html)、[`cli::cli_warn()`](https://cli.r-lib.org/reference/cli_abort.html)、[`cli::cli_inform()`](https://cli.r-lib.org/reference/cli_abort.html)による[cliパッケージ](https://cli.r-lib.org/)への委任です。これらのラッパーはcli書式化が有効であり、洗練された段落の折り返しと行頭記号によるインデント処理で長文の読みやすさを向上させます。次のコードは箇条書き中の長い`!`項目がインデントされた改行により分割されています。

```r
rlang::global_entrace(class = "errorr")
#> Error in `rlang::global_entrace()`:
#> ! `class` must be one of "error", "warning", or "message",
#>   not "errorr".
#> i Did you mean "error"?
```

cliラッパーは他にも補間、テキスト要素の意味的書式化、複数化等の機能を提供します。

```r
inform_marbles <- function(n_marbles) {
  cli::cli_inform(c(
    "i" = "鞄に{n_marbles}個の輝く大理石{?達}を入れています。",
    "v" = "よくやった、{.code cli::cli_inform()}！"
  ))
}

inform_marbles(1)
#> i 鞄に1個の輝く大理石を入れています。
#> v よくやった`cli::cli_inform()`！

inform_marbles(2)
#> i 鞄に2個の輝く大理石達を入れています。
#> v よくやった`cli::cli_inform()`！
```

### `abort()`から`cli_abort()`への移行

大量の`abort()`を[`cli::cli_abort()`](cli_abort.html)に書き換える場合、ユーザーの入力を用いたエラーメッセージの構築に注意してください。入力がcliやglueのシンタックスを含む場合、デバッグの難しいエラーや[予期しない動作](https://xkcd.com/327/)を起こす可能性があります。

```r
user_input <- "{base::stop('Wrong message.', call. = FALSE)}"
cli::cli_abort(sprintf("Can't handle input `%s`.", user_input))
#> Error:
#> ! ! Could not evaluate cli `{}` expression: `base::stop('Wrong...`.
#> Caused by error: 
#> ! Wrong message.
```

上記のエラーや予期しない動作を回避するには、入力の構築にcliを用いてエラーメッセージを保護します。

```r
user_input <- "{base::stop('Wrong message.', call. = FALSE)}"
cli::cli_abort("Can't handle input {.code {user_input}}.")
#> Error:
#> ! Can't handle input `{base::stop('Wrong message.', call. = FALSE)}`.
```

### cliによる書式化のグローバルな有効化

あなたの名前空間におけるすべての`abort()`でcliの書式化を有効にするには、パッケージの`onLoad`フックで`local_use_cli()`を呼び出します。次のコードは`on_load()`を用いた例です（フックで`run_on_load()`を呼び出してください）。

```r
on_load(local_use_cli())
```

`abort()`でのcli書式化を有効にすると次の点で便利です。

- `abort()`から[`cli::cli_abort()`](https://cli.r-lib.org/reference/cli_abort.html)へ段階的に移行する。
- 補間シンタックスを無効化したい場合に`abort()`を利用する。
- `error_cnd()`を用いてエラー状態を作成する。これらの状態メッセージはcliでも自動的に書式化されます。

{% endraw %}
