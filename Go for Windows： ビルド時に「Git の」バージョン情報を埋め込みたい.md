---
title: Go for Windows: ビルド時に「Git の」バージョン情報を埋め込みたい
tags: Go:1.3 Git:1.8 nmake:10.0
author: zetamatta
slide: false
---
* [Go言語: ビルド時にバージョン情報を埋め込みたい](http://qiita.com/suin/items/d643a0ccb6270e8e3734)

よーし、おじさんも、Git のリビジョン番号を埋め込んじゃうぞー！

```mf:Makefile
build :
	for /F %%V IN ('git log -1 --pretty^=format:%%H') DO go build -ldflags "-X main.version [SNAPSHOT-%DATE:/=%-%%V]"
```

すまない、**nmake** 以外は帰ってくれないか？

* これで `main.version` という変数に `[SNAPSHOT-(ビルド日付)-(gitのリビジョン)]` という値が設定される
* git のオプションは`--pretty=format:%H` なんだけど、`%`で囲まれた範囲に `=` があると置換だと CMD.EXE が判断してしまうので、`^` で `=` をエスケープする。

なお、Go側のコードについては、[元記事](http://qiita.com/suin/items/d643a0ccb6270e8e3734)を読んだ方が身のためだ。

まぁ、それにしても Go タグがついてるけど、Go のコード全然出てこないね！どちらかというとバッチ芸…

以上


