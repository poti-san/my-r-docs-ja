---
parent: rlang1.1.1-docs
title: 二重評価問題
---

# 二重評価問題

メタプログラミング固有の問題に一見一度しか評価されないコード断片の複数回評価があります。一つの入力から二つの要約を与える次のデータマスク関数を見てください。

```r
summarise_stats <- function(data, var) {
  data %>%
    dplyr::summarise(
      mean = mean({{ var }}),
      sd = sd({{ var }})
    )
}

summarise_stats(mtcars, cyl)
#> # A tibble: 1 x 2
#>    mean    sd
#>   <dbl> <dbl>
#> 1  6.19  1.79
```

この関数はユーザーが単一の列名を与えれば完全に機能します。しかし、データマスク済み引数には*計算*を含められます。

```
summarise_stats(mtcars, cyl * 100)
#> # A tibble: 1 x 2
#>    mean    sd
#>   <dbl> <dbl>
#> 1  619.  179.
```

計算は遅い、あるいは副効果をもたらす場合があります。これらの理由から、計算はコードに現れた回数だけ実行されるべきです（グループ化済みデータフレームのグループ単位計算のように明文化されていない限り）。次のより複雑な計算を見てみましょう。

```r
times100 <- function(x) {
  message("時間のかかる処理...")
  Sys.sleep(0.1)

  message("そしてメッセージのような副効果！")
  x * 100
}

summarise_stats(mtcars, times100(cyl))
#> 時間のかかる処理...
#> そしてメッセージのような副効果！
#> 時間のかかる処理...
#> そしてメッセージのような副効果！
#> # A tibble: 1 x 2
#>    mean    sd
#>   <dbl> <dbl>
#> 1  619.  179.
```

副効果と長い実行時間により、`summarise_stats()`が入力を二度評価したことが明確です。これは異なる二か所にディフューズ済み式を注入したために起きています。データマスク済み式は次のような行を作成します（キャレット記号「^」はquosure境界を表します）。

```r
dplyr::summarise(
  mean = ^mean(^times100(cyl)),
  sd = ^sd(^times100(cyl))
)
```

この`times100(cyl)`式は二度評価されますが、コードには一度しか現れません。これが二重評価バグです。簡単な解決策のひとつはディフューズ済み入力の定数への割り当てです。コード中の定数は次のコードを参照してください。

```r
summarise_stats <- function(data, var) {
  data %>%
    dplyr::transmute(
      var = {{ var }},
    ) %>%
    dplyr::summarise(
      mean = mean(var),
      sd = sd(var)
    )
}
```

上のコードではディフューズ済み入力は一度だけ評価されます。注入が一度だけだからです。

```r
summarise_stats(mtcars, times100(cyl))
#> 時間のかかる処理...
#> そしてメッセージのような副効果！
#> # A tibble: 1 x 2
#>    mean    sd
#>   <dbl> <dbl>
#> 1  619.  179.
```

## グルー文字列とは？

グルー文字列の`{{`囲いは二重評価問題に遭遇しません。

```r
summarise_stats <- function(data, var) {
  data %>%
    dplyr::transmute(
      var = {{ var }},
    ) %>%
    dplyr::summarise(
      "mean_{{ var }}" := mean(var),
      "sd_{{ var }}" := sd(var)
    )
}

summarise_stats(mtcars, times100(cyl))
#> 時間のかかる処理...
#> そしてメッセージのような副効果！
#> # A tibble: 1 x 2
#>   `mean_times100(cyl)` `sd_times100(cyl)`
#>                  <dbl>              <dbl>
#> 1                 619.               179.
```

グルー文字列は式の結果を必要としないため、元のコードを文字列に変換する（deparseする）だけです。注入式は評価しません。
