---
parent: rlang1.1.1 トピックス
title: データマスク処理とは何でなぜ`{{`が必要なのか？
---

{% raw %}

# データマスク処理とは何でなぜ`{{`が必要なのか？

データマスク処理はRの独特な機能であり、列を通常のオブジェクトとして定義することでデータセット上の直接的なプログラミング処理を可能とします。

```r
# 非データマスクプログラミング
mean(mtcars$cyl + mtcars$am)
#> [1] 6.59375

# カラム参照エラー - データが見つかりません。
mean(cyl + am)
#> Error:
#> ! object 'cyl' not found

# データマスク処理
with(mtcars, mean(cyl + am))
#> [1] 6.59375
```

データマスク処理はデータフレームを用いた対話的プログラミングを簡単にする一方、関数の作成を難解にします。データマスク済み引数を関数へ渡すにはエンブレース演算子`{{`やより複雑な場合は注入演算子`!!`による注入が必要です。

## データマスク処理にはエンブレースと注入が必須ですか？

注入（あるいは疑似引用）はプログラムの一部を変更するメタプログラミング機能です。この機能はRコードの即時評価を防ぐディフューズのために内部のデータマスク処理処理で必要です。ディフューズ済みコードはデータフレームの列定義よりも後のコンテキストで再開されます。次のコードは通常の方法で`summarise()`のようなデータマスク処理関数に引数を渡した場合の動作を示します。

```r
my_mean <- function(data, var1, var2) {
  dplyr::summarise(data, mean(var1 + var2))
}

my_mean(mtcars, cyl, am)
#> Error in `dplyr::summarise()`:
#> i In argument: `mean(var1 + var2)`.
#> Caused by error:
#> ! object 'cyl' not found
```

上のコードの問題は`summarise`による与えられたRコード、即ち`mean(var1 + var2)`のディフューズです。求める動作は`mean(cyl + am)`です。これは注入が必要となる理由です。関数に与えられたコードを`var1`と`var2`の代わりに注入してコードの一部を変更する必要があります。

データマスクコンテキストにおける関数引数の注入は`{{`で囲うだけです。

```r
my_mean <- function(data, var1, var2) {
  dplyr::summarise(data, mean({{ var1 }} + {{ var2 }}))
}

my_mean(mtcars, cyl, am)
#> # A tibble: 1 x 1
#>   `mean(cyl + am)`
#>              <dbl>
#> 1             6.59
```

データマスク処理関数周りの関数作成を詳細に学ぶには「データマスクプログラミングパターン」を参照してください。

## 「マスク処理」の意味は？

通常のRプログラミングオブジェクトは現在の環境、具体的にはグローバル環境や関数の環境に定義されます。

```r
factor <- 1000

# ここでは`factor`を計算に使用できます。
mean(mtcars$cyl * factor)
#> [1] 6187.5
```

この環境は現在のスコープのすべての関数を含みます。スクリプト内は[`library()`](https://rdrr.io/r/base/library.html)によりアタッチされた関数を含み、パッケージ内では他のパッケージからインポートされた関数を含みます。評価がデータフレームだけで実行された場合、計算に必要なオブジェクトや関数は見失われます。

スコープ内のオブジェクトと関数を保持するため、データフレームは環境群の現連鎖の末尾に挿入されます。データフレームは最優先となり、ユーザー環境よりも優先されます。言い換えればユーザー環境を「マスク」します。

マスク処理はデータを優先してユーザー環境と混和します。それにより本当はローカルオブジェクトを使いたいのにRはデータフレームのカラムを使うことがあります。

```r
# 環境変数を定義します。
cyl <- 1000

# データ変数を参照します。
dplyr::summarise(mtcars, mean(cyl))
#> # A tibble: 1 x 1
#>   `mean(cyl)`
#>         <dbl>
#> 1        6.19
```

tidy評価フレームワークはデータマスクとユーザーコンテキストの曖昧さをなくす代名詞を提供します。コードを記述時にこの代名詞を使用することはしばしば良い方法です。

```r
cyl <- 1000

mtcars %>%
  dplyr::summarise(
    mean_data = mean(.data$cyl),
    mean_env = mean(.env$cyl)
  )
#> # A tibble: 1 x 2
#>   mean_data mean_env
#>       <dbl>    <dbl>
#> 1      6.19     1000
```

詳細は[データマスクの曖昧さ](topic-data-mask-ambiguity.md)を参照してください。

## データマスク処理はどのように機能する？

データマスク処理は次の3つの言語機能に依存します。

- baseの[`substitute()`](https://rdrr.io/r/base/substitute.html)、rlangの`enquo()`、`enquos()`、`{{`による引数ディフューズ。Rコードはディフューズされ、データの追加された特殊な環境で遅延評価できます。

- 第一級環境。環境はディフューズ済みRコードを評価できるリスト様オブジェクトの特殊型です。環境中の名前付き要素はオブジェクトを定義します。リストとデータフレームは環境に変換できます。

  ```r
  as.environment(mtcars)
  #> <environment: 0x7febb17e3468>
  ```

- base の[`eval()`](https://rdrr.io/r/base/eval.html)やrlangの`eval_tidy()`による明示的評価。Rコードがディフューズされるとき、評価は中断されます。再開は[`eval()`](https://rdrr.io/r/base/eval.html)により可能です。

  ```r
  expr(1 + 1)
  #> 1 + 1

  eval(expr(1 + 1))
  #> [1] 2
  ```

  既定では[`eval()`](https://rdrr.io/r/base/eval.html)と`eval_tidy()`は現在の環境で評価します。

  ```r
  code <- expr(mean(cyl + am))
  eval(code)
  #> Error:
  #> ! object 'am' not found
  ```

  オプションで環境に変換できるリストやデータフレームを供給できます。

  ```r
  eval(code, mtcars)
  #> [1] 6.59375
  ```

ディフューズ済みコードはデータマスクコンテキストで評価されます。

## 履歴

tidyverseはggplot2のようなパッケージでデータマスクの手法を取り入れ、最終的にrlangパッケージによる固有のプログラミングフレームワークを開発しました。SとRの作者、それに続くランドマーク開発がなければいずれも開発できませんでした。

- S言語で[`attach()`](https://rdrr.io/r/base/attach.html)によるデータスコープが導入されました（Becker, Chambers and Wilks, The New S Language, 1988）。
- S言語で関数モデル作成にデータマスク式が導入されました（Chambers and Hastie, 1993）。
- RチームのPeter Dalgaardが1997年にフレームツールパッケージを記述しました。後の[`base::transform()`](https://rdrr.io/r/base/transform.html)、[`base::subset()`](https://rdrr.io/r/base/subset.html)としてRに含まれています。このAPIはdplyrパッケージのインスピレーションの重要な源です。これは「選択」の初期形態であり、[tidyselect パッケージ](https://tidyselect.r-lib.org/articles/syntax.html)で拡張・成文化されたデータマスクのバリアントです。
- RチームのLuke Tierneyが式をオリジナル環境の軌跡を保持するように変更しました（[changed formulas](https://github.com/wch/r-source/commit/a945ac8e)）。R 1.1.0で公開されたこの変更は衛生的なデータマスク処理、即ちオリジナル環境におけるシンボルの正確な解決の決定的な一歩でした。
- Lukeが2001年に[`base::with()`](https://rdrr.io/r/base/with.html)を導入しました。
- [data.tableパッケージ](https://r-datatable.com)が2006年にデータフレームに対する`[`メソッドの`i`および`j`引数にデータマスク処理と選択を含めました。
- [dplyrパッケージ](https://dplyr.tidyverse.org/)が2014年に公開されました。
- rlangパッケージがtidyverseのデータマスク処理フレームワークとして2017年にtidy評価を開発しました。quosure表記、`!!`と`!!!`による暗黙の注入、データ代名詞が導入されました。
- [rlang 0.4.0](https://www.tidyverse.org/blog/2019/06/rlang-0-4-0/)が2019年にディフューズ＆注入パターンを簡単にする`{{`注入を導入しました。この演算子はRプログラマーがより直感的かつ最小限の定型文で関数にデータマスク済み引数を転送できるようにします。

## 関連項目

- [データマスクプログラミングパターン](topic-data-mask-programming.md)
- [R式のディフューズ](topic-defuse.md)

{% endraw %}
