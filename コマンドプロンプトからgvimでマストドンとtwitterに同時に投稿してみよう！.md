---
title: コマンドプロンプトからgvimでマストドンとtwitterに同時に投稿してみよう！
tags: mastodon Twitter Go gvim バッチファイル
author: zetamatta
slide: false
---
必要なソフトウェア
================

* [twty (A command-line twitter client)](https://github.com/mattn/twty)
* [go-mastdon](https://github.com/mattn/go-mastodon)
* [gvim](https://www.kaoriya.net/software/vim/)
* ビルド用に [Go](https://golang.org/dl/)

twtyのビルド
===========

```powershell
go get github.com/mattn/twty
cd go\src\github.com\mattn\twty\
go build
copy twty.exe (%PATH%のとおったフォルダー)
```

twty は一度起動して、アカウント登録しておくこと

go-mastodon のビルド
===================

```powershell
go get github.com/mattn/go-mastodon
cd go\src\github.com\mattn\go-mastodon\
go build
cd cmd\mstdn\
go build
copy mstdn.exe (%PATH%のとおったフォルダー)
```

* もしかしたら、追加モジュールがいるかもしれないけど、適宜 go get お願いします
* こちらも、いちど mstdn.exe を起動して、アカウント登録してください

バッチファイル用意
================

```powershell:dondon.cmd
@echo off
setlocal
set "DON=%TEMP%\don.txt"
type nul > "%DON%"
gvim -c "e ++enc=utf8" "%DON%"
if errorlevel 1 exit /b
twty -ff "%DON%"
mstdn toot -ff "%DON%"
endlocal
```

コマンドプロンプトから`dondon` とタイプすると gvim が起動するので、魂の叫びを書き込もう。登録する時は普通に「`:wq`」、キャンセルする時は「`:cq`」だ。文字数超過で twitter への投稿だけが失敗する時もあるから注意な！

