---
parent: rlang 1.1.1 トピックス
title: メタプログラミングパターン
---

{%raw%}

# メタプログラミングパターン

この記事のカバーするパターンは「メタプログラミング」、即ちR式のディフューズ・作成・展開・注入機能に依存します。R言語プログラミングをはじめたばかりなら[Advanced R](https://adv-r.hadley.nz)の[Metaprogramming chapter](https://adv-r.hadley.nz/metaprogramming.html)を推奨します。

理解を早めるために先ずは[データマスクプログラミングパターン](topic-data-mask-programming.html)を読んでください。より少ない理論しか要求しない単純なパターンを扱っています。引数の動作や様々なパターン（転送・名前・ブリッジ・変形パターン）といった概念を道具箱に追加できます。

## 転送パターン

### ディフューズと注入

`{{`と`...`はほとんどの用途で使えます。しかし、転送操作からディフューズと注入という2つの構成手順への分解が必要な場合もあります。

`{{`は`enquo()`と`!!`の組み合わせです。従って、以下の関数は完全に同等です。

```r
my_summarise <- function(data, var) {
  data %>% dplyr::summarise({{ var }})
}
my_summarise <- function(data, var) {
  data %>% dplyr::summarise(!!enquo(var))
}
```

`...`に渡すことは`enquos()`と`!!!`の組み合わせと同等です。

```r
my_group_by <- function(.data, ...) {
  .data %>% dplyr::group_by(...)
}
my_group_by <- function(.data, ...) {
  .data %>% dplyr::group_by(!!!enquos(...))
}
```

ディフューズと注入を分解する利点はディフューズ済み式へのアクセスです。一度ディフューズすれば対象とするコンテキストに注入する前に式を調査や変更できます。

### 入力ラベルの調査

ここでは`as_label()`を使ってディフューズ済み引数を自動命名する方法を具体的に学びます。

```r
f <- function(var) {
  var <- enquo(var)
  as_label(var)
}

f(cyl)
#> [1] "cyl"

f(1 + 1)
#> [1] "1 + 1"
```

このコードは`englue()`による引数の書式化と本質的に同等です。

```r
f2 <- function(var) {
  englue("{{ var }}")
}

f2(1 + 1)
#> [1] "1 + 1"
```

複数引数には複数変数の`enquos()`を使います。`.named`を`TRUE`に設定すると名前指定の無い入力は`as_label()`で自動命名します（ほとんどのdplyr関数も同様に動作します）。

```r
g <- function(...) {
  vars <- enquos(..., .named = TRUE)
  names(vars)
}

g(cyl, 1 + 1)
#> [1] "cyl"   "1 + 1"
```

[`dplyr::mutate()`](https://dplyr.tidyverse.org/reference/mutate.html)と同じく、使用者は名前を明示的に指定して自動命名を上書きできます。

```r
g(foo = cyl, bar = 1 + 1)
#> [1] "foo" "bar"
```

ディフューズ＆注入パターンは入力の変形で最も便利です。いくつかの適用例は変形パターンセクションで紹介しています。

## 名前パターン

### シンボル化と注入

シンボル化＆注入パターンは`across(all_of()`を使えない場合に使う名前パターンです。名前を容れたベクトルを表すデータ変数を参照するディフューズ済み式の作成からなります。式はデータマスクコンテキストに注入されます。

単一文字列は`sym()`や`data_sym()`でシンボル化します。

```r
var <- "cyl"

sym(var)
#> cyl

data_sym(var)
#> .data$cyl
```

文字列ベクトルは`syms()`や`data_syms()`でシンボル化します。

```r
vars <- c("cyl", "am")

syms(vars)
#> [[1]]
#> cyl
#> 
#> [[2]]
#> am

data_syms(vars)
#> [[1]]
#> .data$cyl
#> 
#> [[2]]
#> .data$am
```

`sym()`や`syms()`の返す単純なシンボルは（特に一部のbase関数で）幅広く使えますが、ほとんどの場合は`data_sym()`や`data_syms()`を使うでしょう。後者はより頑強だからです（詳細は[データマスクの曖昧さ](topic-data-mask-ambiguity.md)参照）。これらは「シンボル」そのものを返さないことに注意してください。代わりに`.data`代名詞をサブセット化する`$`の「呼び出し」を作成します。

`.data`代名詞はtidy評価機能なのでbase関数では使用できません。原則として、tidy評価関数への注入では`data_`接頭辞を持つ方、base関数への注入では接頭辞を持たない方を使います。

シンボルのリストはスプライス演算子`!!!`でデータマスク済みドットに注入できます。リストの各要素は個別に与えられた引数として注入されます。例えば列名の文字列ベクトルを取る`group_by()`変種の実装は次のように書けます。

```r
my_group_by <- function(data, vars) {
  data %>% dplyr::group_by(!!!data_syms(vars))
}

my_group_by(vars)
```

より複雑な場合にはシンボルの周りにRコードを追加したいでしょう。これには変形パターンが必要です。下のセクションを見てみましょう。

## ブリッジパターン

### ブリッジ選択データマスクとしての`mutate()`

これは[データマスクプログラミングパターン](topic-data-mask-programming.md)で記載された`transmute()`ブリッジパターンの中間段階で`...`を実体化させない変種です。代わりに`...`式はディフューズと調査が可能です。`mutate()`はこの（列というより）式をスプライスします。

```r
my_pivot_longer <- function(data, ...) {
  # ドットをディフューズして名前を調査します。
  dots <- enquos(..., .named = TRUE)
  names <- names(dots)

  # 入力を`mutate()`に渡します。
  data <- data %>% dplyr::mutate(!!!dots)

  # `all_of()`により名前で`...`の入力を選択します。
  data %>%
    tidyr::pivot_longer(cols = all_of(names))
}

mtcars %>% my_pivot_longer(cyl, am = am * 100)
```

1. `...`式をディフューズします。`.named`引数は無名の入力に`mutate()`に渡したときのような既定名を保証します。入力のリストの名前ベクトルが得られます。

2. 名前ベクトルを入手したら`mutate()`の引数式に注入してデータフレームを更新します。

3. 最後に名前ベクトルを[`all_of()`](https://tidyselect.r-lib.org/reference/all_of.html)でtidy選択に渡します。

## 変形パターン

### 入力の手動変形

`across()`と変種が使えない場合、メタプログラミングのテクニックを使って自分で入力を変形することになります。このテクニックを示すため、`across()`を使わずに`my_mean()`を再実装してみましょう。このパターンは入力式のディフューズ、その周囲のより大きな呼び出しの組み立て、データマスク関数への変更済み式の注入からなります。

簡単のために単一名前付き引数から始めてみましょう。

```r
my_mean <- function(data, var) {
  # 式をディフューズする。
  var <- enquo(var)

  # `mean()`の呼び出しにラップする。
  var <- expr(mean(!!var, na.rm = TRUE))

  # 拡張した式を注入する。
  data %>% dplyr::summarise(mean = !!var)
}

mtcars %>% my_mean(cyl)
#> # A tibble: 1 x 1
#>    mean
#>   <dbl>
#> 1  6.19
```

`...`を使う場合も類似ですが、少し複雑になります。複数変種の`enquos()`と`!!!`を使います。また、[`purrr::map()`](https://purrr.tidyverse.org/reference/map.html)による可変個数入力のループも使います。しかし、それ以外は基本的に上の場合と同様です。

```r
my_mean <- function(.data, ...) {
  # ドットをディフューズします。自動命名を有効化します。
  vars <- enquos(..., .named = TRUE)

  # ディフューズ式をそれぞれ写像して、`mean()`呼び出しでラップします。
  vars <- purrr::map(vars, ~ expr(mean(!!.x, na.rm = TRUE)))

  # 式を注入します。
  .data %>% dplyr::summarise(!!!vars)
}

mtcars %>% my_mean(cyl)
#> # A tibble: 1 x 1
#>     cyl
#>   <dbl>
#> 1  6.19
```

`summarise()`のデータマスク動作の継承に注意してください。`...`を関数の内側へ効率的に転送しているからです。これはtidy選択動作を継承する`across()`に基づく変形パターンとの違いです。実用的には関数の選択ヘルパーや構文への非対応を意味します。代わりに新しいベクトルを臨機応変に作成する能力が得られます。

```r
mtcars %>% my_mean(cyl = cyl * 100)
#> # A tibble: 1 x 1
#>     cyl
#>   <dbl>
#> 1  619.
```

## baseパターン

このセクションでは「base」データマスク関数を用いたプログラミングパターンをレビューします。これらは本質的にデータマスク中の式組み立て・評価からなります。パターンをレビューしてrlangの作法と比べてみましょう。

### データマスク済み[`get()`](https://rdrr.io/r/base/get.html)

baseパターンの最も簡単な版は、データマスクからオブジェクトを取得するための変数名を伴った[get()](https://rdrr.io/r/base/get.html)呼び出しです。

```r
var <- "cyl"

with(mtcars, mean(get(var)))
#> [1] 6.1875
```

この種類のパターンには名前衝突の隙があります。例えば入力データフレームが`var`という名前の変数を持つ場合です。

```r
df <- data.frame(var = "wrong")

with(df, mean(get(var)))
#> Error in `get()`:
#> ! object 'wrong' not found
```

通常、このような衝突を避けるために[`get()`](https://rdrr.io/r/base/get.html)よりもシンボル注入が推奨されます。base関数を使う場合、`inject()`の明示的な使用による注入演算子の有効化が必要です。

```r
inject(
  with(mtcars, mean(!!sym(var)))
)
#> [1] 6.1875
```

名前衝突の詳細は[データマスクの曖昧さ](topic-data-mask-ambiguity.md)を参照してください。

### データマスク済み[`parse()`](https://rdrr.io/r/base/parse.html)と[`eval()`](https://rdrr.io/r/base/eval.html)

より複雑なパターンは文字列によるRコード組み立てとそのデータマスクにおける評価からなります。

```r
var1 <- "am"
var2 <- "vs"

code <- paste(var1, "==", var2)
with(mtcars, mean(eval(parse(text = code))))
#> [1] 0.59375
```

前の例と同様に`code`変数は名前衝突に脆弱です。さらなる重要事項として、`var1`と`var2`が使用者の入力による場合、[敵対的コード](https://xkcd.com/327/)を含む可能性があります。文字列から組み立てられたコードの評価は常に危険な仕事です。

```r
var1 <- "(function() {
  Sys.sleep(Inf)  # コインマイニング動作かもしれません。
})()"
var2 <- "vs"

code <- paste(var1, "==", var2)
with(mtcars, mean(eval(parse(text = code))))
```

コードを内部のみで使う場合は大きな問題にはなりません。しかし、このコードはインターネット使用者の使うShinyアプリケーションの一部である可能性があります。内部使用の場合も変数名が`-`や`:`のような構文記号を含む場合は解析バグの原因になります。

```r
var1 <- ":var:"
var2 <- "vs"

code <- paste(var1, "==", var2)
with(mtcars, mean(eval(parse(text = code))))
#> Error in `parse()`:
#> ! <text>:1:1: unexpected ':'
#> 1: :
#>     ^
```

これらの理由からコード解析の代わりに「組み立て済み」コードが常に推奨されます。`sym()`による変数名の組み立ては入力のサニタイジング方法です。

```r
var1 <- "(function() {
  Sys.sleep(Inf)  # コインマイニング動作かもしれません。
})()"
var2 <- "vs"

code <- call("==", sym(var1), sym(var2))

code
#> `(function() {\n  Sys.sleep(Inf)  # コインマイニング動作かもしれません。\n})()` == 
#>     vs
```

このとき敵対的入力はエラーを送出します。

```r
with(mtcars, mean(eval(code)))
#> Error:
#> ! object '(function() {\n  Sys.sleep(Inf)  # コインマイニング動作かもしれません。\n})()' not found
```

最後に、名前衝突を避ける手段としてコード注入はコード評価よりも推奨されます。

```r
var1 <- "am"
var2 <- "vs"

code <- call("==", sym(var1), sym(var2))
inject(
  with(mtcars, mean(!!code))
)
#> [1] 0.59375
```
{%endraw%}
