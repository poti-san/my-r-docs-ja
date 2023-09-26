---
parent: rlang1.1.1-docs
title: エラーメッセージに関数呼び出しを含める
---

{% raw  %}
# エラーメッセージに関数呼び出しを含める

rlang 1.0から`abort()`は既定でメッセージ中にエラーを発生した関数を含めます。

```r
my_function <- function() {
  abort("それはできません。")
}

my_function()
#> Error in `my_function()`:
#> ! それはできません。
```

この機能は`abort()`が失敗した関数で直接呼び出された場合にうまく機能します。しかし、`abort()`の呼び出しが他の関数にエクスポートされた場合（いわゆる「エラーヘルパー」）、`abort`がエラーを送出した関数を明示する必要があります。

## ユーザーコンテキストに渡す

エラーヘルパーは2種類あります。

- 単純な`abort()`ラッパー。クラスや属性へのエラー状態の構造的な追加を目的としてしばしば用いられます。
  
  ```r
  stop_my_class <- function(message) {
    abort(message, class = "my_class")
  }
  ```

- 入力チェック関数。入力チェッカーは典型的に入力と引数名を受け取ります。入力が期待と異なる場合、エラーを送出します。
  
  ```r
  check_string <- function(x, arg = "x") {
    if (!is_string(x)) {
      cli::cli_abort("{.arg {arg}}は文字列であるべきです。")
    }
  }
  ```

どちらの場合も既定のエラー呼び出しはエンドユーザーが使いやすいとは言えません。ユーザー関数ではなく内部関数を反映するためです。

```r
my_function <- function(x) {
  check_string(x)
  stop_my_class("未実装")
}
```

```r
my_function(NA)
#> Error in `check_string()`:
#> ! `x`は文字列であるべきです。
```

```r
my_function("foo")
#> Error in `stop_my_class()`:
#> ! 未実装
```

これの修正するため、`call`引数として対応する関数環境を`abort()`に渡してエラーを送出した関数を知らせます。

```r
stop_my_class <- function(message, call = caller_env()) {
  abort(message, class = "my_class", call = call)
}

check_string <- function(x, arg = "x", call = caller_env()) {
  if (!is_string(x)) {
    cli::cli_abort("{.arg {arg}}は文字列であるべきです。", call = call)
  }
}
```

```r
my_function(NA)
#> Error in `my_function()`:
#> ! `x`は文字列であるべきです。
```

```r
my_function("foo")
#> Error in `my_function()`:
#> ! 未実装
```

### 入力チェッカーと`caller_arg()`

`caller_arg()`ヘルパーは異なる関数の入力をチェックする入力チェッカーで便利です。`"x"`がチェックされる引数の名前ではない場合に`arg = "X"`とハードコードして呼び出し側に提供させる代わりに、`caller_arg()`を使用します。

```ｒ
check_string <- function(x,
                         arg = caller_arg(x),
                         call = caller_env()) {
  if (!is_string(x)) {
    cli::cli_abort("{.arg {arg}}は文字列であるべきです。", call = call)
  }
}
```

[`substitute()`](https://rdrr.io/r/base/substitute.html)と`rlang::as_label()`の組み合わせはより使いやすい既定値を提供します。

```r
# 訳者中：サンプルコードが違う
my_function <- function(my_arg) {
  check_string(my_arg)
}

my_function(NA)
#> Error in `my_function()`:
#> ! `my_arg`は文字列であるべきです。
```

## 副次効果：バックトレースのトリミング

`call`として`caller_env()`を渡す別の効果は`abort()`によるエラーヘルパーの自動的な隠匿です。

```r
my_function <- function() {
  their_function()
}
their_function <- function() {
  error_helper1()
}

error_helper1 <- function(call = caller_env()) {
  error_helper2(call = call)
}
error_helper2 <- function(call = caller_env()) {
  if (use_call) {
    abort("それはできません。", call = call)
  } else {
    abort("それはできません。")
  }
}
```

```r
use_call <- FALSE
their_function()
#> Error in `error_helper2()`:
#> ! それはできません。
```

```r
rlang::last_error()
#> <error/rlang_error>
#> Error in `error_helper2()`:
#> ! それはできません。
#> ---
#> Backtrace:
#>     x
#>  1. \-rlang (local) their_function()
#>  2.   \-rlang (local) error_helper1()
#>  3.     \-rlang (local) error_helper2(call = call)
#> Run rlang::last_trace(drop = FALSE) to see 1 hidden frame.
```

正しい`call`があればバックトレースはより簡単になり、ユーザーはスタックの関連する部分にフォーカスできます。

```r
use_call <- TRUE
their_function()
#> Error in `their_function()`:
#> ! それはできません。
```

```r
rlang::last_error()
#> <error/rlang_error>
#> Error in `their_function()`:
#> ! それはできません。
#> ---
#> Backtrace:
#>     x
#>  1. \-rlang (local) their_function()
#> Run rlang::last_trace(drop = FALSE) to see 3 hidden frames.
```

## testthatワークフロー

エラーのスナップショットはエラーメッセージ中の正しいエラー呼び出しをチェックする主な方法です。しかし、新しいtestthatによる警告およびエラースナップショット表示の採用が必要です。新しい表示はrlangにより`call`フィールドを含めて出力されます。これにより警告よびエラーメッセージの完全な外観のユーザーに表示されるメッセージとしての監視が簡単になります。

この表示はまだすべてのパッケージには適用されていません。testthat 3.1.2ではオプトインのためにrlang >= 1.0.0へ明示的に依存しています。testthat 3.1.3からはバージョンに関係なくrlangに依存しており、自由にオプトインできます。将来的にはすべてのパッケージで新しい表示が有効になります。

エラースナップショットは一度有効にすれば作成できます。

```r
expect_snapshot(error = TRUE, {
  my_function()
})

expect_snapshot_error(my_function())
```

エラーメッセージのスナップショット被覆率はパッケージの十分さにつながることを改めて確認してください。
<% end raw %>
