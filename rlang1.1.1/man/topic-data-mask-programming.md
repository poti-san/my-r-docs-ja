---
parent: rlang 1.1.1 トピックス
title: データマスクプログラミングパターン
---

{% raw %}

# データマスクプログラミングパターン

データマスク関数を他の関数中で使うには特別なプログラミングパターンが必要です。ここでは特定の問題を解決するためのパターンを説明・比較します。初心者の方は次のどちらかのチュートリアルから始めた方が良いかもしれません。

- [Programming with dplyr](https://dplyr.tidyverse.org/articles/programming.html)
- [Using ggplot2 in packages](https://ggplot2.tidyverse.org/articles/ggplot2-in-packages.html)

ディフューズと注入表現について学びたい場合は「メタプログラミングパターン」を読んでください。

## パターンの選択

主に次の2つの項目をしてプログラミングパターンでデータマスク関数をラップするかを決めます。

1. ラップされる関数の実装する動作は？
2. あなたの関数の実装する動作は？

これらの質問への回答により、以下のアプローチを選びます。

- **転送パターン**ではあなたの関数は連動する関数の動作を継承します。
- **名前パターン**ではあなたの関数は列名を示す文字列や文字列ベクトルを受け取ります。
- **ブリッジパターン**では引数を継承する代わりに引数の動作を変更します。

単一の名前付き引数では`...`内の複数の引数とは異なる解決策を用います。

## 引数の動作

通常の関数では、引数は受け取るオブジェクトの型を単位として定義できます。引数は文字列ベクトル、データフレーム、単一の論理値等を受け取れます。一方、データマスク済み引数はより複雑です。特定の型のオブジェクトを受け取る（例えば[`dplyr::mutate()`](https://dplyr.tidyverse.org/reference/mutate.html)はベクトルを受け取ります）だけでなく、特殊な計算動作を実施します。

- データマスク済み式（base）。例えば[`transform()`](https://rdrr.io/r/base/transform.html)や[`with()`](https://rdrr.io/r/base/with.html)。式は与えられたデータフレームの列を参照できます。
- データマスク済み式（tidy評価）。例えば[`dplyr::mutate()`](https://dplyr.tidyverse.org/reference/mutate.html)やggplot2::aes()。baseのデータマスクと同じですが、tidy評価機能も有効です。`{{`、`!!`のような注入演算子や`.data`、`.env`代名詞です。
- データマスク済みシンボル。データマスク済み引数と同じですが、与えられる式は単純な列名です。これはしばしば物事を簡単にします。具体的には二重評価問題を簡単に回避できます。
- [tidy選択](https://tidyselect.r-lib.org/reference/language.html)。例えば[`dplyr::select()`](https://dplyr.tidyverse.org/reference/select.html)や`tidyr::pivot_longer()`。データマスクの代替であり、`starts_with()`や`all_of()`のような選択ヘルパーをサポートします。[`c()`](https://rdrr.io/r/base/c.html)、`|`、`&`のような演算子の特殊動作も実装します。
  データマスクと異なり、tidy選択は解釈された方言です。実際には何もマスクされません。式はデータフレームのコンテキストでの解釈（例えば列`cyl`と`am`の和集合を意味する`c(cyl, am)`）とユーザー環境での評価（例えば`all_of()`、`starts_with()`、その他の任意の式）のどちらも実施されます。これは以下で示す引数の動作に影響します。
- 動的ドット。データマスク済み引数、tidy選択、通常の引数で使われます。動的ドットではglue演算子による名前注入と`!!!`演算子による複数引数の注入が使えます。

あなたの関数引数のキャパシティーを知らせるには継承するセマンティクスの集合により次のタグで文書化します。

```r
@param foo <[`data-masked`][dplyr::dplyr_data_masking]> What `foo` does.

@param bar <[`tidy-select`][dplyr::dplyr_tidy_select]> What `bar` does.

@param ... <[`dynamic-dots`][rlang::dyn-dots]> What these dots do.
```

## 転送パターン

転送パターンでは引数は代入されたデータマスク済み引数の動作を継承します。

### `{{`による囲い

エンブレース演算子`{{`は単一引数の転送構文です。データマスクコンテキストで引数を転送できます。

```r
my_summarise <- function(data, var) {
  data %>% dplyr::summarise({{ var }})
}
```

tidy選択でも同様です。

```r
my_pivot_longer <- function(data, var) {
  data %>% tidyr::pivot_longer(cols = {{ var }})
}
```

関数は周囲のコンテキストの動作を自動的に継承します。例えば、データマスクコンテキストに転送された引数では列の参照や`.data`代名詞が使えます。

```r
mtcars %>% my_summarise(mean(cyl))

x <- "cyl"
mtcars %>% my_summarise(mean(.data[[x]]))
```

また、tidy選択に転送された引数ではすべてのtidy選択機能を使用できます。

```r
mtcars %>% my_pivot_longer(cyl)
mtcars %>% my_pivot_longer(vs:gear)
mtcars %>% my_pivot_longer(starts_with("c"))

x <- c("cyl", "am")
mtcars %>% my_pivot_longer(all_of(x))
```

### `...`転送

ドットが既に転送構文であるため、`...`引数による単純な転送には特別な構文は不要です。いつものように別の引数に渡せます。データマスク済み引数とも連携できます。

```r
my_group_by <- function(.data, ...) {
  .data %>% dplyr::group_by(...)
}

mtcars %>% my_group_by(cyl = cyl * 100, am)
```

tidy選択とも連携できます。

```r
my_select <- function(.data, ...) {
  .data %>% dplyr::select(...)
}

mtcars %>% my_select(starts_with("c"), vs:carb)
```

いくつかの関数は単一の名前付き引数でtidy選択を受け取ります。次のコードでは[c()](https://rdrr.io/r/base/c.html)の中に`...`を渡しています。

```r
my_pivot_longer <- function(.data, ...) {
  .data %>% tidyr::pivot_longer(c(...))
}

mtcars %>% my_pivot_longer(starts_with("c"), vs:carb)
```

tidy選択の中では[c()](https://rdrr.io/r/base/c.html)はベクトルではなく選択を結合します。これにより`...`を受け取る関数と単一の引数を受け取る引数を簡単に仲介できます。

## 名前パターン

名前パターンでは環境変数中の文字列や文字列ベクトル中の名前で列を参照します。転送パターンは引数を渡す関数の中で主に使われますが、名前パターンはどこでも使えます。

- スクリプトでは`for`や[`lapply()`](https://rdrr.io/r/base/lapply.html)により文字列ベクトルをループ処理でき、名前とデータ変数の接続には`.data`パターンを使用します。tidy選択ヘルパー`all_of()`によりベクトルを一度に与えられます。
- 関数の引数で名前パターンを使えば使用者はデータマスクの複雑性を気にせずに通常のデータ変数名を使用できます。

### `.data`代名詞のサブセット化

`.data`代名詞はtidy評価の機能であり、`{{`のようなすべてのデータマスク済み引数が有効です。代名詞はデータマスクを表し、`[[`や`$`によるサブセット化も可能です。次の3つのコードは同等です。

```r
mtcars %>% dplyr::summarise(mean = mean(cyl))

mtcars %>% dplyr::summarise(mean = mean(.data$cyl))

var <- "cyl"
mtcars %>% dplyr::summarise(mean = mean(.data[[var]]))
```

`.data`代名詞はループ中でもサブセット化できます。

```r
vars <- c("cyl", "am")

for (var in vars) print(dplyr::summarise(mtcars, mean = mean(.data[[var]])))
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1  6.19
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1 0.406

purrr::map(vars, ~ dplyr::summarise(mtcars, mean =  mean(.data[[.x]])))
#> [[1]]
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1  6.19
#> 
#> [[2]]
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1 0.406
```

関数の引数とデータ変数の接続にも使えます。

```r
my_mean <- function(data, var) {
  data %>% dplyr::summarise(mean = mean(.data[[var]]))
}

my_mean(mtcars, "cyl")
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1  6.19
```

実装上、上の`my_mean()`はデータマスク動作から完全に絶縁（訳者注：ディフューズ比喩の仲間？）されており、通常の関数のように呼び出せます。

```r
# マスク無効
am <- "cyl"
my_mean(mtcars, am)
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1  6.19
#訳者中：6.19はmtcars$cylの平均。mtcars$amは参照されていない。

# プログラム可能
my_mean(mtcars, tolower("CYL"))
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1  6.19
```

### 名前の文字列ベクトル

`.data`代名詞は単一の列名でだけサブセット化できます。単一括弧によるインデックス操作には非対応です。

```r
mtcars %>% dplyr::summarise(.data[c("cyl", "am")])
#> Error in `dplyr::summarise()`:
#> i In argument: `.data[c("cyl", "am")]`.
#> Caused by error in `.data[c("cyl", "am")]`:
#> ! `[` is not supported by the `.data` pronoun, use `[[` or $ instead.
#訳：`[`は`.data`代名詞に対応していません。`[[`か$を使ってください。
```

tidy評価には`.data`の複数形バージョンはありません。代わりに`all_of()`演算子を使うことで文字列ベクトルを与えたtidy選択が使用できます。`all_of()`演算子はtidy選択を受け取る関数内で簡単に使えます。次のコードは`tidyr::pivot_longer()`の例です。

```r
vars <- c("cyl", "am")
mtcars %>% tidyr::pivot_longer(all_of(vars))
#> # A tibble: 64 x 11
#>     mpg  disp    hp  drat    wt  qsec    vs  gear  carb name  value
#>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <chr> <dbl>
#> 1    21   160   110   3.9  2.62  16.5     0     4     4 cyl       6
#> 2    21   160   110   3.9  2.62  16.5     0     4     4 am        1
#> 3    21   160   110   3.9  2.88  17.0     0     4     4 cyl       6
#> 4    21   160   110   3.9  2.88  17.0     0     4     4 am        1
#> # i 60 more rows
```

tidy選択に未対応の関数では*ブリッジパターン*を使用できます。ブリッジパターンについては次の項目で説明します。ブリッジが無効または不便な場合、シンボル化＆注入パターンを使った軽いメタプログラミングも使えます。

## ブリッジパターン

呼び出す関数があなたの関数の引数に与えたい動作を実装していないことがあります。

このような関数を使う場合、少しの思案が必要でしょう。ある動作を異なる動作に置き換える体系的な方法がないからです。一般的な方法は求める動作を実装するコンテキストへの引数の転送です。ここでは戻り値を対象動詞または関数にブリッジする方法を探ります。

### 選択からデータマスクへのブリッジ`across()`

dplyr 1.0は`across()`によりすべての動作でtidy選択に対応しています。この関数は通常、複数列のマッピングに使われますが、単一の選択にも使えます。例えば、`group_by()`に引数を渡すとき、データマスクの代わりにtidy選択インターフェイスを使うには`across()`をブリッジとして使用します。

```r
my_group_by <- function(data, var) {
  data %>% dplyr::group_by(across({{ var }}))
}

mtcars %>% my_group_by(starts_with("c"))
```

`across()`は単一引数の選択を受け取ります（複数引数を受け取る`select()`とは違います）が、`...`による直接渡しはできません。代わりに[`c()`](https://rdrr.io/r/base/c.html)を使います。`c()`はtidy選択で複数選択を単一引数として与えるための方法です。

```r
my_group_by <- function(.data, ...) {
  .data %>% dplyr::group_by(across(c(...)}))
}

mtcars %>% my_group_by(starts_with("c"), vs:gear)
```

### データマスクブリッジ用の名前（names）`across(all_of())`

変数を`across()`に転送する代わりに`all_of()`に渡すことで、データマスクブリッジ用の名前（names）を作成できます。

```r
my_group_by <- function(data, vars) {
  data %>% dplyr::group_by(across(all_of(vars)))
}

mtcars %>% my_group_by(c("cyl", "am"))
```

このブリッジテクニックは名前のベクトルをデータマスク済みコンテキストに接続するときに使用します。

### 選択ブリッジ用のデータマスクとしての`transmute`

tidy選択にデータマスク済み引数を渡す方法はもう少しトリッキーで、次の3段階が必要です。

```r
my_pivot_longer <- function(data, ...) {
  # データマスクコンテキストの`...`を`transmute()`で転送します。
  # また入力の名前を保存します。
  inputs <- dplyr::transmute(data, ...)
  names <- names(inputs)

  # データを入力で更新します。
  data <- dplyr::mutate(data, !!!inputs)

  # `all_of()`で入力を名前で選択します。
  # Select the inputs by name with `all_of()`
  tidyr::pivot_longer(data, cols = all_of(names))
}

mtcars %>% my_pivot_longer(cyl, am = am * 100)
```

1. 最初に`...`式を`transmute()`に渡します。`mutate()`とは異なり、ユーザーの入力から新しいデータフレームが作成されます。ここでの目的は`...`中の名前の調査です。これは名前なし引数用に作成された既定の名前を含みます。

2. 名前を手に入れたら、`mutate()`に引数を注入してデータフレームを更新します。

3. 最後に`all_of()`で名前をtidy選択に渡します。

## 変形パターン（トランスフォームパターン）

### 名前付き入力 vs `...`

名前付き引数は簡単に変形できます。単純にRコードの括弧済み入力を囲います。例えば、次の`my_summarise()`関数は単なる`summarise()`と便利さが変わりません。

```r
my_summarise <- function(data, var) {
  data %>% dplyr::summarise({{ var }})
}
```

変数のほかにコードを追加すればより便利になります。

```r
my_mean <- function(data, var) {
  data %>% dplyr::summarise(mean = mean({{ var }}, na.rm = TRUE))
}
```

しかし、`...`による入力にはこのテクニックを使えません。ドット要素のプレースホルダーを用いたRコードの記述にはある種のドット用テンプレート構文が必要です。これはtidy評価には組み込まれていませんが、[`dplyr::across()``](https://dplyr.tidyverse.org/reference/across.html)、[`dplyr::if_all()``](https://dplyr.tidyverse.org/reference/across.html)、[`dplyr::if_any()``](https://dplyr.tidyverse.org/reference/across.html)のような演算子を使えます。それも不可能な場合、式を手動でテンプレート化できます。

### `across()`による入力の変換

dplyrの`across()`演算子は式と入力セットのマッピングに便利です。ここでは`...`で与えられたすべての引数の[`mean()`](https://rdrr.io/r/base/mean.html)を計算する`my_mean`を作成します。最も簡単な方法はドットの`across()`への転送です（これは`...`をtidy選択動作へ継承させます）。

```r
my_mean <- function(data, ...) {
  data %>% dplyr::summarise(across(c(...), ~ mean(.x, na.rm = TRUE)))
}

mtcars %>% my_mean(cyl, carb)
#> # A tibble: 1 x 2
#>     cyl  carb
#>   <dbl> <dbl>
#> 1  6.19  2.81

mtcars %>% my_mean(foo = cyl, bar = carb)
#> # A tibble: 1 x 2
#>     foo   bar
#>   <dbl> <dbl>
#> 1  6.19  2.81

mtcars %>% my_mean(starts_with("c"), mpg:disp)
#> # A tibble: 1 x 4
#>     cyl  carb   mpg  disp
#>   <dbl> <dbl> <dbl> <dbl>
#> 1  6.19  2.81  20.1  231.
```

### `if_all()`と`if_any()`を用いた入力の変形

[`dplyr::filter()``](https://dplyr.tidyverse.org/reference/filter.html)は`across()`とは異なる演算子を要求します。`&`や`|`による論理式の組み合わせを必要とするためです。この問題を解決するためにdplyrは`across()`のバリアントとして`if_all()`と`if_any()`を導入しました。次のコードは、変数セットの最小値と一致しないすべての行をフィルターします。

```r
filter_non_baseline <- function(.data, ...) {
  .data %>% dplyr::filter(if_all(c(...), ~ .x != min(.x, na.rm = TRUE)))
}

mtcars %>% filter_non_baseline(vs, am, gear)
```
{% endraw %}
