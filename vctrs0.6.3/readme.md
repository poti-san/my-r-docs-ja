---
title: vctrs 0.6.3 readme
---
{% raw %}
# vctrs 0.6.3

- [原文](https://vctrs.r-lib.org/)

vctrsパッケージには主な３つの目的があります（いずれもvignetteの記載通り）。

- [`vec_size()`](https://vctrs.r-lib.org/reference/vec_size.html)と[`vec_ptype()`](https://vctrs.r-lib.org/reference/vec_ptype.html)による[`length()`](https://rdrr.io/r/base/length.html)と[`class()`](https://rdrr.io/r/base/class.html)の代替（[`vignette("type-size")`](https://vctrs.r-lib.org/articles/type-size.html)）。これらはサイズ再利用および型強制のフレームワークと組み合わせて定義されています。`ptype`はプロトタイプ（何かの原型や象徴的な形式）の概念を連想させます。

- サイズ＆型安定を望ましい関数特性として定義するため、既存のbase関数の分析やより良い代替の提案に用いる（[`vignette("stability")`](https://vctrs.r-lib.org/articles/stability.html)）。これは特に[`c()`](https://rdrr.io/r/base/c.html)、[`ifelse()`](https://rdrr.io/r/base/ifelse.html)、[`rbind()`](https://rdrr.io/r/base/cbind.html)に理想的な特性を与えるモチベーションに基づきます。

- S3ベクトルの作成を簡単にする新しい`vctr`既定クラス（[`vignette("s3-vector")`](https://vctrs.r-lib.org/articles/s3-vector.html)）。vctrsは少し新しいvctrsジェネリクスの観点――実装を大幅に単純＆より頑強にする――で多くのbaseジェネリクスにメソッドを提供します。

vctrsは開発者にフォーカスしたパッケージです。vctrsの理解や拡張には開発者の努力が必要ですが、ほとんどの利用者からは見えません。私たちの願いは基礎となる理論によってユーザーが理論の厳密な学習抜きで正しいメンタルモデルを組み立てられることです。vctrsは通常他のパッケージから使用されて、tidyverse等のサポートする新しいS3ベクトルクラスの提供を簡単にします。以上の理由から、vctrsは限定的な依存関係しか持ちません。

## インストール

CRANからインストールするには次のようにします。

```r
install.packages("vctrs")
```

開発版をインストールするには次のようにします。

```r
# install.packages("pak")
pak::pak("r-lib/vctrs")
```

## 使い方

```r
library(vctrs)

# 大きさ
str(vec_size_common(1, 1:10))
#>  整数10
str(vec_recycle_common(1, 1:10))
#> List of 2
#>  $ : num [1:10] 1 1 1 1 1 1 1 1 1 1
#>  $ : int [1:10] 1 2 3 4 5 6 7 8 9 10

# プロトタイプ
str(vec_ptype_common(FALSE, 1L, 2.5))
#>  整数0
str(vec_cast_common(FALSE, 1L, 2.5))
#> List of 3
#>  $ : num 0
#>  $ : num 1
#>  $ : num 2.5
```

## モチベーション

vctrsの最初のモチベーションは2つの仕切られた、しかし関係した問題から生まれました。最初の問題は[`base::c()`](https://rdrr.io/r/base/c.html)が異なるS3ベクトルを混ぜる動作の不適切さです

```r
# ファクターの連結は整数を作ります。
c(factor("a"), factor("b"))
#> [1] 1 1

# 日付と日時の連結は不正な値を作ります。結合順も問題となります。
dt <- as.Date("2020-01-01")
dttm <- as.POSIXct(dt)

c(dt, dttm)
#> [1] "2020-01-01"    "4321940-06-07"
c(dttm, dt)
#> [1] "2019-12-31 19:00:00 EST" "1970-01-01 00:04:22 EST"
```

この動作は[`c()`](https://rdrr.io/r/base/c.html)が二重の目的を持つことに由来します。ひとつめはベクトルの連結。ふたつめは属性の除去です。例えば、[`?POSIXct`](https://rdrr.io/r/base/DateTimeClasses.html)はタイムゾーンのリセットに[`c()`](https://rdrr.io/r/base/c.html)を用いることを推奨します。

二つ目の問題は[`dplyr::bind_rows()`](https://dplyr.tidyverse.org/reference/bind_rows.html)が外部から拡張不可能なことです。現在、`dplyr::bind_rows()`はヒューリスティックによって任意のS3クラスを扱います。しかし、これはしばしば失敗するため、合理的な解決策を組み立てる必要を感じました。これはデータフレームのリスト・データフレーム・行列を含む、データフレーム列の多くの型をサポートする明確な必要と交差します。
{% endraw %}