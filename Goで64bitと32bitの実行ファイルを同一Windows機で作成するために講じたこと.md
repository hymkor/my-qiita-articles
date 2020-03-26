---
title: Goで64bitと32bitの実行ファイルを同一Windows機で作成するために講じたこと
tags: Go
author: zetamatta
slide: false
---
ビルド時の必須作業
================

まず、コンパイラは 64bit バージョンをインストールすることにした。ただし、ライブラリも 64bit だけになってしまう。これに対してはビルド時の警告で親切に教えてくれるとおり：

```
cd "%GOROOT%"
cd src
set GOARCH=386
make.bat
```

で 32bit 版のライブラリも用意してやればよい。以後、

- 64bit 版のビルドは、そのままでよい
- 32bit 版のビルドは `set GOARCH=386` と環境変数を設定してやればよい

バイナリがごっちゃにならない工夫
=============================

ただ、これだけだと、作った32bit版と64bit版がごっちゃになってしまう恐れがあるので、以下のような対策を打った。

ソースディレクトリをを分ける
-------------------------

- 64bit版のソースのフォルダー(例：nyagos64) と、32bit版のソースのフォルダー(nyagos32)を別途用意。内容は git で同期
- `go build` を直接使わず、make.cmd というビルド用のバッチを用意する（これは前からそうしていた）。で、make.cmd の頭で

```
setlocal
if exist goarch.txt for /F %%I in (goarch.txt) do set "GOARCH=%%I"
```

というコマンドを追加しておく。32bit版のフォルダーにだけ「386」だけ書いた、goarch.txt を置いてやれば、32bit版のフォルダーでビルドした場合は自動で 32bit バージョンになる。

配布用の zip ファイル作成
-----------------------

また、配布用 zip ファイルを作る時も

```
setlocal
if exist goarch.txt for /F %%I in (goarch.txt) do set "GOARCH=%%I"
if "%GOARCH%" == "" for /F "delims=/ tokens=2" %%I in ('go version') do set "GOARCH=%%I"
zip -9 "nyagos-%VERSION%-%GOARCH%.zip" nyagos.exe …
```

と zip ファイル名の末尾に %GOARCH% をつけてやればバッチリ。
（%GOARCH% が分からない時は `go version` で go 自身に聞けばよい）

実行ファイル自身がアーキテクチャを分かっていたら便利
-----------------------------------------------

`runtime.GOARCH` という変数に「386」「amd64」というアーキテクチャ文字列(string)が入るので、それをそのまま用ればよい。起動時にどこかに表示しておきたい。

その他
------

- プログラムが起動したらタスクマネージャでプロセスを見る。32bitプロセスは「*32」という文字列がイメージ名の末尾につく
- 外部の DLL の類も 32bit と 64bit それぞれそろえなくてはいけない。間違えると実行時に panic になることが多い（たとえば、`lua53.dll` … LuaBinaries の DLL、正直 64bit版は `lua53x64.dll` とか適当に名前を変えてほしかった）
- EXEファイル自体のビット数を確認するのに、[APEC](https://bitbucket.org/yamorijp/apec/wiki/Home)というツールがありました。

以上

