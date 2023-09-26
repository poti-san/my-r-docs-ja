---
parent: rlang1.1.1-docs
title: 状態メッセージのカスタマイズ
---

{% raw %}

# 状態メッセージのカスタマイズ

[cli パッケージ](https://cli.r-lib.org)のオプションを使用して`abort()`、`warn()`、`inform()`の表示する状態メッセージの様々な部分をカスタマイズできます。

## 箇条書きのUnicode行頭文字の無効化

箇条書きは既定で Unicode シンボルを接頭辞に用います。

```r
rlang::abort(c(
  "The error message.",
  "*" = "Regular bullet.",
  "i" = "Informative bullet.",
  "x" = "Cross bullet.",
  "v" = "Victory bullet.",
  ">" = "Arrow bullet."
))
#> Error:
#> ! The error message.
#> • Regular bullet.
#> ℹ Informative bullet.
#> ✖ Cross bullet.
#> ✔ Victory bullet.
#> → Arrow bullet.
```

このオプションを使用して簡単な文字に置換できます。

```r
options(cli.condition_unicode_bullets = FALSE)

rlang::abort(c(
  "The error message.",
  "*" = "Regular bullet.",
  "i" = "Informative bullet.",
  "x" = "Cross bullet.",
  "v" = "Victory bullet.",
  ">" = "Arrow bullet."
))
#> Error:
#> ! The error message.
#> * Regular bullet.
#> i Informative bullet.
#> x Cross bullet.
#> v Victory bullet.
#> > Arrow bullet.
```

## 行頭文字の変更

cliユーザーテーマで各行頭記号を設定できます。例えば次のコードはすべての行頭記号を`*`にします。

```r
options(cli.user_theme = list(
  ".cli_rlang .bullet-*" = list(before = "* "),
  ".cli_rlang .bullet-i" = list(before = "* "),
  ".cli_rlang .bullet-x" = list(before = "* "),
  ".cli_rlang .bullet-v" = list(before = "* "),
  ".cli_rlang .bullet->" = list(before = "* ")
))

rlang::abort(c(
  "The error message.",
  "*" = "Regular bullet.",
  "i" = "Informative bullet.",
  "x" = "Cross bullet.",
  "v" = "Victory bullet.",
  ">" = "Arrow bullet."
))
#> Error:
#> ! The error message.
#> * Regular bullet.
#> * Informative bullet.
#> * Cross bullet.
#> * Victory bullet.
#> * Arrow bullet.
```

`bullett`クラスを使えば先導を含むすべての行頭文字を同じ記号にできます。

```r
options(cli.user_theme = list(
  ".cli_rlang .bullet" = list(before = "* ")
))

rlang::abort(c(
  "The error message.",
  "*" = "Regular bullet.",
  "i" = "Informative bullet.",
  "x" = "Cross bullet.",
  "v" = "Victory bullet.",
  ">" = "Arrow bullet."
))
#> Error:
#> * The error message.
#> * Regular bullet.
#> * Informative bullet.
#> * Cross bullet.
#> * Victory bullet.
#> * Arrow bullet.
```

## エラー呼び出しの前景色と背景色の変更

関数内で`abort()`を呼び出すとエラーのコンテキストを示すために関数呼び出しを表示します。

```r
splash <- function() {
  abort("Can't splash without water.")
}

splash()
#> Error in `splash()`:
#> ! Can't splash without water.
```

関数呼び出しはcliパッケージにより`code`要素として書式化されます。ここでは示しませんが、コード文字列は既定で背景色を強調して書式化されます。これは確実な検出を目的としており、背景色は使用するテーマの明暗によって変わります。cliテーマのコード要素の色は上書きできます。次のコードは私がIDEで使用しているカラーテーマに適した私的な設定です。

```r
options(cli.user_theme = list(
  span.code = list(
    "background-color" = "#3B4252",
    color = "#E5E9F0"
  )
))
```

{% endraw %}
