---
parent: rlang 1.1.1 トピックス
title: quosureとは何でいつ必要か？
---

{% raw %}
# quosureとは何でいつ必要か？

quosureはディフューズ済み式用の特殊な型で、式の書かれたコンテキストの追跡を保持します。quosureの追跡能力はデータマスク関数とのインターフェイス接続で重要です。関数は2つの異なるパッケージのような無関係な環境に含まれる場合があるためです。

## 環境の混合

具体例を見てみましょう。ここではR使用者はBMI値の統計量を保持するデータフレームを要約するためにfooパッケージの`summarise_bmi()`関数を呼び出すとします。このデータフレームの`height`変数はメートル単位ではないので、`div100()`カスタム関数を使って列のスケールを変更します。

```r
# ユーザーのグローバル環境

div100 <- function(x) {
  x / 100
}

dplyr::starwars %>%
  foo::summarise_bmi(mass, div100(height))
```

`summarise_bmi()`は以下のようにfooパッケージの名前空間で定義されたデータマスク関数です。

```r
# fooパッケージの名前空間

bmi <- function(mass, height) {
  mass / height^2
}

summarise_bmi <- function(data, mass, height) {
  data %>%
    bar::summarise_stats(bmi({{ mass }}, {{ height }}))
}
```

fooパッケージは2つのベクトルの計算に`bmi()`カスタム関数を使います。また、別のパッケージbarで定義された`summarise_stats()`をインターフェイス接続します。barは以下のように名前空間が異なります。

```r
# barパッケージの名前空間

check_numeric <- function(x) {
  stopifnot(is.numeric(x))
  x
}

summarise_stats <- function(data, var) {
  data %>%
    dplyr::transmute(
      var = check_numeric({{ var }})
    ) %>%
    dplyr::summarise(
      mean = mean(var, na.rm = TRUE),
      sd = sd(var, na.rm = TRUE)
    )
}
```

今後はbarパッケージの`check_numeric()`カスタム関数を使って入力を検証します。dplyrパッケージのデータマスク関数ともインターフェイス接続します（二重評価問題を避けるためにdefine-a-constantトリックを使います）。

以下のコード断片では3つのデータマスク関数が同時にインターフェイス接続されています。

- 下部の[`dplyr::transmute()`](https://dplyr.tidyverse.org/reference/transmute.html)はデータマスク済み入力を受け取り、`var`という単一列を持つデータフレームを作成します。
- その手前の`bar::summarise_stats()`は[`dplyr::transmute()`](https://dplyr.tidyverse.org/reference/transmute.html)内部のデータマスク済み入力を受け取り、数値か判定します。
- 最初の`foo::summarise_bmi()``bar::summarise_stats()`内の2つのデータマスク済み入力を受け取り、それらをBMI値に変換します。

4つめのコンテキストはグローバル環境であり、`summarise_bmi()`がデータフレームに定義された2つの列を使って呼び出されるコンテキストです。列の片方は`div100()`ユーザー関数によりその場で変形されています。

すべてのコンテキスト（グローバル環境のいくらかは除き）はプライベートで外部の関数から見えない関数を含みます。今のところ、行として評価される最終展開されたデータマスク済み式は次のようになります（キャレット`^`はquosure境界を示します）。

```r
dplyr::transmute(
  var = ^check_numeric(^bmi(^mass, ^div100(height)))
)
```

quosureの役割は`check_numeric()`はbarパッケージ、`bmi()`はfooパッケージ、`div100()`はグローバル環境で見つかるとRに伝えることです。

## いつquosureを作るべき？

tidyverseを使うとき、quosureについて悩む必要はありません。`{{`や`...`が用意してくれるからです。[Programming with dplyr](https://dplyr.tidyverse.org/articles/programming.html)や[データマスクプログラミングパターン](topic-data-mask-programming.md)のような入門テキストはquosureという用語を言及さえしません。より複雑な場合、`enquo()`や`enquos()`を使ってquosureを作成する必要があるでしょう（その場合もそれらの関数がquosureを返すことは知る必要がありません）。ここではquosureのより高度な適用方法を探索します。

### 外部とローカルの式

大まかにいえばquosureは`enquo()`や`enquos()`でディフューズされた引数でのみ必要です（または暗黙に`enquo()`を呼び出す`{{`でも）。

```r
my_function <- function(var) {
  var <- enquo(var)
  their_function(!!var)
}

# 同等です。
my_function <- function(var) {
  their_function({{ var }})
}
```

quosureによるディフューズ済み引数のラップが必要なのは、引数として与えられる式が異なる環境、即ちユーザー環境から与えられるためです。自作関数内で作成したローカル式ではquosureの作成が通常は不要です。

```r
my_mean <- function(data, var) {
  # `expr()`は`quo()`不要です。
  expr <- expr(mean({{ var }}))
  dplyr::summarise(data, !!expr)
}

my_mean(mtcars, cyl)
#> # A tibble: 1 x 1
#>   `mean(cyl)`
#>         <dbl>
#> 1        6.19
```

`quo()`は`expr()`の代わりに使えますが、過剰使用です。[‘dplyr::summarise()`](https://dplyr.tidyverse.org/reference/summarise.html)は`enquos()`を使い、呼び出した環境のquosureで与えた式をラップする責任を持つからです。

手動評価する場合も同様です。既定では[`eval()`](https://rdrr.io/r/base/eval.html)と`eval_tidy()`は呼び出し側の環境を捕捉します。

```r
my_mean <- function(data, var) {
  expr <- expr(mean({{ var }}))
  eval_tidy(expr, data)
}

my_mean(mtcars, cyl)
#> [1] 6.1875
```

### 外部ディフューズ

上記の例外（自前の式ではなくて外部の式をquosureでラップするとき）は関数が`...`の代わりにリストで複数式を受け取る場合です。この場合の推奨方法はtidy選択を受け取り、使用者に複数列を[‘c()`]([c: Combine Values into a Vector or List](https://rdrr.io/r/base/c.html))で結合させることです。それができない場合、外部ディフューズ済み式のリストを受け取ります。

```r
my_group_by <- function(data, vars) {
  stopifnot(is_quosures(vars))
  data %>% dplyr::group_by(!!!vars)
}

mtcars %>% my_group_by(dplyr::vars(cyl, am))
```

このパターンでは[‘dplyr::vars()‘](https://dplyr.tidyverse.org/reference/vars.html)が式を外部ディフューズしています。この関数はquosureのリストを作成します。式は通常の引数のように関数から関数へ渡されるからです。実際には[`dplyr::vars()`](https://dplyr.tidyverse.org/reference/vars.html)と`ggplot2::vars()`は`quos()`の単純な別名です。

```r
dplyr::vars(cyl, am)
#> <list_of<quosure>>
#> 
#> [[1]]
#> <quosure>
#> expr: ^cyl
#> env:  global
#> 
#> [[2]]
#> <quosure>
#> expr: ^am
#> env:  global
```

外部ディフューズについて詳細な情報を得るには[`...`を使わずに複数列を受け取る](topic-multiple-columns.md)を参照してください。

## quosureの技術解説

quosureは次の2つを持ちます。

- 式（`quo_get_expr()`で取得できます）。
- 環境（`quo_get_env()`で取得できます）。

加えて以下の動作を実装します。

- 「呼び出し可能」。評価により結果を生産します。
  歴史的な事情により[`base::eval()`](https://rdrr.io/r/base/eval.html)はquosue評価に対応しません。現在、quosueには`eval_tidy()`が必要です。将来的にはこの限界を修正したいと考えています。
- 「衛生的」。追跡した環境で評価されます。
- 「マスク可能」。データマスク内（現在、`eval_tidy()`または`new_data_mask()`で）で評価される場合、データマスクはquosure環境手前のスコープに入ります。
- 概念上、quosureはデータマスクとユーザー環境という2つの連鎖を環境から継承します。実用的には、rlangはデータマスクの一番上に現在評価中のquosure環境を連鎖させることでこの特殊なスコープ処理を実装しています。

Promise（promisesパッケージの非同期式ではなくてRが遅延評価に使う方）とquosureは似ています。重要な違いはPromiseは一度だけ評価されて以降のために結果をキャッシュすることです。quosureは呼び出しの度に動作して繰り返し評価されます。

## 関連項目

- `enquo()`と`enquos()`は関数の引数をquosureとしてディフューズします。これらはquosureを作成する主な方法です。
- `quo()`は`expr()`に似ていますが、quosureでラップされます。通常、自前のローカル式はラップ不要です。
- `quo_get_expr()`と`quo_get_env()`でquosureの成分にアクセスできます。
- `new_quosure()` と`as_quosure()`で部分からquosureを構築できます。

{% endraw %}
