---
parent: "rlang 1.1.1 トピックス"
title: "`...`を使わずに複数列を受け取る"
---

{% raw %}

# `...`を使わずに複数列を受け取る

ここでは単一の関数引数で複数列を受け取る方法を比較します。

おさらいとして、データマスク済み関数に引数を渡す通常の方法は2つあります（詳細は[# データマスクプログラミングパターン](topic-data-mask-programming.md)参照）。単一の引数は`{{`で囲います。

```r
my_group_by <- function(data, var) {
  data %>% dplyr::group_by({{ var }})
}

my_pivot_longer <- function(data, var) {
  data %>% tidyr::pivot_longer({{ var }})
}
```

`...`中の複数の引数は`group_by()`のような`...`を受け取る関数ならそのまま渡し、`pivot_longer()`のような単一の引数でtidy選択を受け取る関数では[`c()`](https://rdrr.io/r/base/c.html)で渡します。

```r
# ドットをそのまま渡します。
my_group_by <- function(.data, ...) {
  .data %>% dplyr::group_by(...)
}

my_pivot_longer <- function(.data, ...) {
  .data %>% tidyr::pivot_longer(c(...))
}
```

ところで`...`ではなく単一の名前付き引数に複数列を渡す場合はどうするのでしょうか？

## tidy選択の使用

単一の引数に複数列を与えるtidyverseの慣用的な方法は「tidy選択」です（引数の動作セクション参照）。tidy選択で単一の引数に複数列を渡す構文は[`c()`](https://rdrr.io/r/base/c.html)です。

```r
mtcars %>% tidyr::pivot_longer(c(am, cyl, vs))
```

`{{`は動作を継承するため、`my_pivot_longer()`の実装は複数列を与える操作に自動で対応します。

```r
my_pivot_longer <- function(data, var) {
  data %>% tidyr::pivot_longer({{ var }})
}

mtcars %>% my_pivot_longer(c(am, cyl, vs))
```

データマスク済み引数を受け取る`group_by()`ではブリッジとして`across()`を使えます（ブリッジパターンを参照）。

```r
my_group_by <- function(data, var) {
  data %>% dplyr::group_by(across({{ var }}))
}

mtcars %>% my_group_by(c(am, cyl, vs))
```

tidy選択コンテキストの波括弧囲いも`across()`も不可能な場合、[`tidyselect::eval_select()`](https://tidyselect.r-lib.org/reference/eval_select.html)によるtidy選択動作を手動で実装すべきでしょう。

## 外部ディフューズの使用

引数にtidy選択動作を実装するには引数のディフューズが必要です。しかし、歴史的に通常の引数として動作する引数のディフューズ対応は破壊的変更です。 代替案は引数の外部ディフューズの使用です。例えばformulaインターフェイスが挙げられます。モデリング関数は通常引数にformulaを受け取りformulaはユーザーコードをディフューズします。

```r
my_lm <- function(data, f, ...) {
  lm(f, data, ...)
}

mtcars %>% my_lm(disp ~ drat)
```

formulaのディフューズ済み表現は一度作成すれば通常引数のように関数へ渡せます。`facet_`関数は同様の方法でtidy評価に対応しています。`vars()`関数（`quos()`の単純な別名）はユーザーがそれらの引数を外部でディフューズするために提供されています。

```r
ggplot2::facet_grid(
  ggplot2::vars(cyl),
  ggplot2::vars(am, vs)
)
```

単純にディフューズ済み式のリストを引数として与えればこの方法を実装できます。このリストはそのようなリストを受け取る他の関数へ通常の方法で渡せます。

```r
my_facet_grid <- function(rows, cols, ...) {
  ggplot2::facet_grid(rows, cols, ...)
}
```

`!!!`によるスプライスも使えます。

```r
my_group_by <- function(data, vars) {
  stopifnot(is_quosures(vars))
  data %>% dplyr::group_by(!!!vars)
}

mtcars %>% my_group_by(dplyr::vars(cyl, am))
```

## # ノン・アプローチ：リストの解析

直感的に、単一引数に式のリストを与えようとするほとんどのプログラマーは引数をディフューズして解析しようとします。ユーザーは[`list()`](https://rdrr.io/r/base/list.html)表現に複数引数を与えることが期待されます。そのような呼び出しを見つけた場合、引数は回収されて`!!!`で接合されます。それ以外の場合、ユーザーは`!!`で注入された単一引数の供給を仮定されます。このような実装は次のようになります。

```r
my_group_by <- function(data, vars) {
  vars <- enquo(vars)

  if (quo_is_call(vars, "list")) {
    expr <- quo_get_expr(vars)
    env <- quo_get_env(vars)
    args <- as_quosures(call_args(expr), env = env)
    data %>% dplyr::group_by(!!!args)
  } else {
    data %>% dplyr::group_by(!!vars)
  }
}
```

これは単純な例では動作します。

```r
mtcars %>% my_group_by(cyl) %>% dplyr::group_vars()
#> [1] "cyl"

mtcars %>% my_group_by(list(cyl, am)) %>% dplyr::group_vars()
#> [1] "cyl" "am"
```

しかし、この方法は以下の限界にすぐぶつかります。

```r
mtcars %>% my_group_by(list2(cyl, am))
#> Error in `group_by()`: Can't add columns.
#> i `..1 = list2(cyl, am)`.
#> i `..1` must be size 32 or 1, not 2.
```

複数列の解析ではtidy選択構文の[`c()`](https://rdrr.io/r/base/c.html)を使ったインターフェイスで一貫した方が良いでしょう。通常はtidy選択か外部ディフューズどちらかの使用を推奨します。

{% endraw %}
