---
title: here 1.0.1 readme
---

{% raw %}

# here 1.0.1 (r-lib)

- [原文](https://here.r-lib.org/)

hereパッケージの目的はプロジェクト指向ワークフローにおけるファイル参照を簡単にすることです。[`setwd()`](https://rdrr.io/r/base/getwd.html)にはディレクトリ構成への依存という危うさがありますが、hereはプロジェクトの最上位ディレクトリを使うことでファイルパスの組み立てを容易にします。

# インストール

```r
install.packages("here")
```

# 使い方

hereパッケージは最上位ディレクトリからの相対パスを作成します。読み込みまたは`here()`実行時に現在のプロジェクトの最上位ディレクトリを表示します。

```r
here::i_am("README.Rmd")
#> here() starts at /home/kirill/git/R/here
here()
#> [1] "/home/kirill/git/R/here"
```

次の方法でファイル入出力用に最上位ディレクトリからの相対パスを組み立てられます。

```r
here("inst", "demo-project", "data", "penguins.csv")
#> [1] "/home/kirill/git/R/here/inst/demo-project/data/penguins.csv"
readr::write_csv(palmerpenguins::penguins, here("inst", "demo-project", "data", "penguins.csv"))
```

相対パスは関連付けられたソースファイルがプロジェクト内のどこにあっても機能します。プロジェクトが異なるサブディレクトリにデータとレポートを持つ分析プロジェクトでもです。実例は同梱の[デモプロジェクト](https://github.com/r-lib/here/tree/master/inst/demo-project)を参照してください。

# リファレンス

## セットアップ

- `i_am()` - 現在のスクリプトまたはレポートの場所の設定

## ファイル位置特定

- `here()` - ファイルの検索

## 診断

- `dr_here()` - 状況報告

{% endraw %}