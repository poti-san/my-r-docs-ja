---
parent: rlang1.1.1-docs
title: なぜ文字列やその他の定数は空の環境でquoseされるのか？
---

{% raw %}
# なぜ文字列やその他の定数は空の環境でquoseされるのか？

関数の引数はquosureにディフューズされ、quosureはディフューズ済み式の環境の追跡を保持します。

```r
quo(1 + 1)
#> <quosure>
#> expr: ^1 + 1
#> env:  global
```

定数を与えた場合、quosureは現在の環境の代わりに空の環境を追跡することに注意してください。

```r
quos("foo", 1, NULL)
#> <list_of<quosure>>
#>
#> [[1]]
#> <quosure>
#> expr: ^"foo"
#> env:  empty
#>
#> [[2]]
#> <quosure>
#> expr: ^1
#> env:  empty
#>
#> [[3]]
#> <quosure>
#> expr: ^NULL
#> env:  empty
```

これは関数の引数から定数の環境の一貫した捕捉が非可能であるRコードのコンパイル処理に拠ります。引数のディフューズは引数を遅延評価するRのPromiseメカニズムに依存します。関数がコンパイルされてRが引数を定数と気づいたとき、関数評価を遅くするPromiseの作成は回避されます。その代わりに関数はPromiseでラップされた定数ではなくて剥き出しの定数を直接与えられます。

## コンパイルによるPromiseラップ解除の具体例

`findVar()`Cレベル関数呼び出しでPromiseを捕捉して最適化を観察できます。

```r
# Promiseの評価を実行せずに`arg`へバインドされたオブジェクトを返します。
f <- function(arg) {
  rlang:::find_var(current_env(), sym("arg"))
}

# シンボルまたは定数に対して`f()`を呼び出す。
g <- function(symbolic) {
  if (symbolic) {
    f(letters)
  } else {
    f("foo")
  }
}

# 上の小さな関数をコンパイル済みにする。
f <- compiler::cmpfun(f)
g <- compiler::cmpfun(g)
```

`f()`がシンボル引数で呼び出されたとき、Rの作成したPromiseオブジェクトを取得できます。

```r
g(symbolic = TRUE)
#> <promise: 0x7ffd79bac130>
```

しかし、定数を与えると定数がそのまま返されます。

```r
g(symbolic = FALSE)
#> [1] "foo"
```

Promiseが無い場合、引数の元の環境を把握する方法はありません。

## 定数の環境は必要？

tidyverseのデータマスクAPIは定数の環境を必要としないように意図して設計されています。

- データマスクAPIは定数を解釈できるべきです。それにより、これまで見てきたように通常の引数を渡して、あるいは`!!`による注入を渡しても呼び出せます。`dplyr::mutate(mtcars, var = cyl)`と`dplyr::mutate(mtcars, var = !!mtcars$cyl)`に差異があってはなりません。
- データマスクは*評価*イディオムであって、*内観*イディオムではありません。データマスク関数の動作は定数（または与えられた値を評価するシンボル）の与えられた呼び出し環境に依存しないべきです。
{% endraw %}
