---
parent: rlang 1.1.1 トピックス
title: エラー連鎖にコンテキスト情報を含める
---

{% raw %}

# エラー連鎖にコンテキスト情報を含める

エラー連鎖はエラー発生時にコンテキスト情報を提供するための仕組みです。次のような多様な状況でエラーの原因や発生源の素早い理解に便利なコンテキストを提供できます。

- 低レベルエラーの発生した高レベルコンテキストの言及。例えば低レベルHTTPエラーから高レベルダウンロードエラーへの連鎖。
- ユーザーエラーの起きたパイプライン段階の言及。tidyverseの多くのNSE（非標準評価）インターフェイスで使用されます。具体的にはdplyr、tidymodels、ggplot2等です。
- ユーザーエラーの発生した「反復コンテキスト」の言及。例えばドキュメント処理中の入力ファイルやループ中のユーザーコード実行時の数値やキーの反復処理です。

以下にdplyrによる連鎖エラーの実例を示します。ユーザーのミスにより、パイプライン段階（`mutate()`）と関数の呼び出された反復コンテキスト（グループID）が表示されます。

```r
add <- function(x, y) x + y

mtcars |>
  dplyr::group_by(cyl) |>
  dplyr::mutate(new = add(disp, "foo"))
#> Error in `dplyr::mutate()`:
#> i In argument: `new = add(disp, "foo")`.
#> i In group 1: `cyl = 4`.
#> Caused by error in `x + y`:
#> ! non-numeric argument to binary operator
```

すべてのケースで2種類のエラーがお互いに連鎖して機能します。

1. **原因エラー（causal error）**：現在の処理を中断します。
2. **コンテキストエラー（contextual error、文脈エラー）**：悪い物事の高レベル情報を表現します。

エラー連鎖には１つ以上のコンテキストエラーが含まれますが、原因エラーは常に1個だけです。

## エラーの再送出

エラー連鎖を作成するため、まずは原因エラーの発生を補足する必要があります。[`tryCatch()`](https://rdrr.io/r/base/conditions.html)や[`withCallingHandlers()`](https://rdrr.io/r/base/conditions.html)よりも`try_fetch()`の使用を推奨します。

- [`tryCatch()`](https://rdrr.io/r/base/conditions.html)とは異なり、`try_fetch()`はエラーのコンテキストを完全に保持します。これはユーザーに完全なバックトレース（例えば`last_error()`によって）を保証し、`options(error = recover)`によってより深いエラーコンテキストへの到達を可能とするため、デバッグ時に重要な機能です。
- [`withCallingHandlers()`](https://rdrr.io/r/base/conditions.html)もエラーコンテキストを保持しますが、`try_fetch()`はR 4.2.0以降でスタックオーバーフローエラーを捕捉できます。

実用的には`try_fetch()`は[`tryCatch()`](https://rdrr.io/r/base/conditions.html)と似たように機能します。エラーのクラス名とハンドルした関数のペアを与えます。エラーを連鎖させるには`parent`引数にそのエラーを渡して単純にエラーハンドラーからエラーを再送出します。

次の例では`with_`関数を作成しています。関数は入力として与えられたコードの実行前にいくつかの設定（ここでは連鎖エラー）をセットアップします。

```r
with_chained_errors <- function(expr) {
  try_fetch(
    expr,
    error = function(cnd) {
      abort("段階中のエラーです。", parent = cnd)
    }
  )
}

with_chained_errors(1 + "")
#> Error in `with_chained_errors()`:
#> ! 段階中のエラーです。
#> Caused by error in `1 + ""`:
#> ! non-numeric argument to binary operator
```

通常、このエラーヘルパーはユーザーが目にする別の関数で使います。

Typically, you'll use this error helper from another user-facing function.

```r
my_verb <- function(expr) {
  with_chained_errors(expr)
}

my_verb(add(1, ""))
#> Error in `with_chained_errors()`:
#> ! 段階中のエラーです。
#> Caused by error in `x + y`:
#> ! non-numeric argument to binary operator
```

連鎖エラーは作成しましたが、コンテキストエラーの呼び出しは不正確です。ユーザーが目にする関数の名前ではなくエラーヘルパーの名前に言及しています。

エラーメッセージに含まれる関数呼び出しを読んだら`abort()`に`call`引数を渡す必要を察するでしょう。これはエラー呼び出しとバックトレースの問題を修正するために必要なことです。

```r
with_chained_errors <- function(expr, call = caller_env()) {
  try_fetch(
    expr,
    error = function(cnd) {
      abort("段階中のエラーです。", parent = cnd, call = call)
    }
  )
}
```

`call`引数で呼び出し環境を渡したので、`abort()`は実行フレームから対応する関数呼び出しを自動選択します。

```r
my_verb(add(1, ""))
#> Error in `my_verb()`:
#> ! 段階中のエラーです。
#> Caused by error in `x + y`:
#> ! non-numeric argument to binary operator
```

### 欠損引数についての捕捉

`my_verb()`は遅延評価パターンを実装します。ユーザー入力はエラー連鎖コンテキストが構成されるまで未評価のまま維持されます。このお膳立てのマイナス面は引数欠損エラーの間違ったコンテキストでの報告です。

```r
my_verb()
#> Error in `my_verb()`:
#> ! 段階中のエラーです。
#> Caused by error in `my_verb()`:
#> ! argument "expr" is missing, with no default
```

修正には連鎖エラーコンテキストを構成する前にこれらの引数を単純に要求します。例えばrlangのエクスポートする`check_required()`入力チェッカーが使えます。

```r
my_verb <- function(expr) {
  check_required(expr)
  with_chained_errors(expr)
}

my_verb()
#> Error in `my_verb()`:
#> ! `expr` is absent but must be supplied.
```

## 原因エラーの完全な所有権取得

原因エラーの所有権を完全に取得してよりユーザーフレンドリーなエラーメッセージと共に再送出できます。この場合、元のエラーはエンドユーザーから完全に隠されます。連鎖でなくこの方法を選ぶときは注意深く考慮すべきです。原因エラーの隠匿はゆーざーが　正確からデバッグ情報を奪う可能性があります。

- 通常、この方法による「ユーザーエラー（dplyr入力等）」の隠匿は悪いアイデアです。
- 低レベルエラーの隠匿は妥当でしょう。例えば低レベルのHTTPエラーを高レベルのダウンロードエラーで置換する場合です。同様にdplyrのようなtidyverseパッケージは低レベルvctrsエラーを自作の高レベルエラーで置換します。
- 見境ない原因エラーの隠匿は避けるべきでしょう。予期しないエラーの情報を抑制し得るからです。通常、非連鎖エラーの再送出は特定のエラークラスでのみ実施されるべきです。

連鎖を使わないエラーの再送出とユーザー視点からの原因エラーの完全な取得のためには、`try_fetch()`でエラーを取得して新しいエラーを送出します。連鎖エラーの送出との唯一の違いは`parent`引数への`NA`の設定です。`parent`引数の無視もできますが、`NA`を渡すことでエラーがハンドラーから再送出されており対応するエラーヘルパーをバックトレースから隠匿すべきことを`abort()`に伝えられます。

```r
with_own_scalar_errors <- function(expr, call = caller_env()) {
  try_fetch(
    expr,
    vctrs_error_scalar_type = function(cnd) {
      abort(
        "ベクトルを与えてください。",
        parent = NA,
        error = cnd,
        call = call
      )
    }
  )
}

my_verb <- function(expr) {
  check_required(expr)
  with_own_scalar_errors(
    vctrs::vec_assert(expr)
  )
}

my_verb(env())
#> Error in `my_verb()`:
#> ! ベクトルを与えてください。
```

低レベルエラーの高レベルエラーオブジェクトへの収納は良い習慣です。デバッグ時の調査が可能となります。次のコードでは低レベルエラーを`error`フィールドに収納しています。以下に`last_error()`の返すオブジェクトのサブセットから元のエラーを参照する方法を示します。

```r
rlang::last_error()$error
#> <error/vctrs_error_scalar_type>
#> Error in `my_verb()`:
#> ! `expr` must be a vector, not an environment.
#> ---
#> Backtrace:
#>     x
#>  1. \-rlang (local) my_verb(env())
```

## ケーススタディ：連鎖エラーの写像化

連鎖エラーの良い使用例として入力集合ループ時の反復状態情報の追加が挙げられます。これを描くため、反復エラーを捕捉した任意のユーザーエラーと連鎖する`map()`／[`lapply()`](https://rdrr.io/r/base/lapply.html)を実装してみます。

以下は`map()`の最低限の実装です。

```r
my_map <- function(.xs, .fn, ...) {
  out <- new_list(length(.xs))

  for (i in seq_along(.xs)) {
    out[[i]] <- .fn(.xs[[i]], ...)
  }

  out
}

list(1, 2) |> my_map(add, 100)
#> [[1]]
#> [1] 101
#> 
#> [[2]]
#> [1] 102
```

この実装ではユーザーはエラー発生時に反復の失敗理由を把握できません。

```r
list(1, "foo") |> my_map(add, 100)
#> Error in `x + y`:
#> ! non-numeric argument to binary operator
```

### 反復情報を伴うエラーの再送出

上記を改善するため、ループを`try_fetch()`呼び出しでラップして反復情報を伴うエラーを再送出します。パフォーマンスへの膨大な打撃を避けるため、`try_fetch()`はループの外側で呼び出してください。

```r
my_map <- function(.xs, .fn, ...) {
  out <- new_list(length(.xs))
  i <- 0L

  try_fetch(
    for (i in seq_along(.xs)) {
      out[[i]] <- .fn(.xs[[i]], ...)
    },
    error = function(cnd) {
      abort(
        sprintf("%d番目の要素を写像中に問題が生じました。", i),
        parent = cnd
      )
    }
  )

  out
}
```

再送出ハンドラーの作成したエラー連鎖はユーザーに失敗した反復の番号を提供できます。

```r
list(1, "foo") |> my_map(add, 100)
#> Error in `my_map()`:
#> ! 2番目の要素を写像中に問題が生じました。
#> Caused by error in `x + y`:
#> ! non-numeric argument to binary operator
```

### 写像関数から送出されたエラーの処理

`my_map()`に提供された関数でエラーが直接発生した場合、ユーザーエラー呼び出しはあまり有益ではありません。

```r
my_function <- function(x) {
  if (!is_string(x)) {
    abort("`x`が文字列ではありません。")
  }
}

list(1, "foo") |> my_map(my_function)
#> Error in `my_map()`:
#> ! 1番目の要素を写像中に問題が生じました。
#> Caused by error in `.fn()`:
#> ! `x`が文字列ではありません。
```

関数自体は名前がありません。関数を参照する変数だけが名前を持ちます。この場合、写像された関数は`.fn`変数へ引数として渡されています。エラー発生時、ユーザーには`.fn`変数の名前が報告されます。

解決案のひとつはエラーの`call`フィールドの調査です。`.fn`呼び出しを検出したら`.fn`引数をディフューズ済みコードに置換します。

```r
my_map <- function(.xs, .fn, ...) {
  # `.fn`に与えられたディフューズ済みコードを捕捉します。
  fn_code <- substitute(.fn)

  out <- new_list(length(.xs))

  for (i in seq_along(.xs)) {
    try_fetch(
      out[[i]] <- .fn(.xs[[i]], ...),
      error = function(cnd) {
        # `call`フィールドを調査して`.fn`呼び出しを検出します。
        if (is_call(cnd$call, ".fn")) {
          # `.fnをディフューズ済みコードに置換します。
          # 既存の引数は維持します。
          cnd$call[[1]] <- fn_code
        }
        abort(
          sprintf("%s番目の要素を写像中に問題が生じました。", i),
          parent = cnd
        )
      }
    )
  }

  out
}
```

出来上がりです。

```r
list(1, "foo") |> my_map(my_function)
#> Error in `my_map()`:
#> ! 1番目の要素を写像中に問題が生じました。
#> Caused by error in `my_function()`:
#> ! `x`が文字列ではありません。
```
{% endraw %}
