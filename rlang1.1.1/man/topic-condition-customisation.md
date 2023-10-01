---
parent: rlang1.1.1 トピックス
title: 状態メッセージのカスタマイズ
---

{% raw %}

# 状態メッセージのカスタマイズ

`abort()`、`warn()`、`inform()`による状態メッセージは[cli パッケージ](https://cli.r-lib.org)のオプションでカスタマイズできます。

## 箇条書きのUnicode行頭文字の無効化

既定では箇条書きの行頭記号はUnicodeシンボルが使われます。

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

次のオプションを使用して行頭記号を簡単な文字に変更できます。

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

## 行頭記号の変更

cliユーザーテーマを使えば行頭記号を設定できます。次のコードはすべての行頭記号を`*`にします。

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

`bullett`クラスを使えばリードを含むすべての行頭記号を同じ記号に統一できます。

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

## エラー呼び出しの前景色・背景色の変更

関数内で`abort()`を呼び出した場合、エラーのコンテキストを示すために関数呼び出しが表示されます。

```r
splash <- function() {
  abort("Can't splash without water.")
}

splash()
#> Error in `splash()`:
#> ! Can't splash without water.
```

関数呼び出しはcliパッケージにより`code`要素として書式化されます。ここでは示しませんが、コード文字列の背景色は既定で強調して書式化されます。これは確実な検出を目的としており、背景色は使用するテーマの明暗に合わせられます。cliテーマのコード要素の色は上書きできます。次のコードは私がIDEで使用しているカラーテーマに適した設定です。

```r
options(cli.user_theme = list(
  span.code = list(
    "background-color" = "#3B4252",
    color = "#E5E9F0"
  )
))
```

{% endraw %}
