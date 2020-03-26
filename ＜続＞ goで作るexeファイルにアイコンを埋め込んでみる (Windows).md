---
title: <続> goで作るexeファイルにアイコンを埋め込んでみる (Windows)
tags: Go:1.3 MinGW:4.8.1.4
author: zetamatta
slide: false
---
# トピック

* [前回](http://qiita.com/zetamatta/items/4aa36661336a56b95e06)は、[rsrc.exe](https://github.com/akavel/rsrc) というツールで .syso というファイルを作れば Ok であった。
* が、MinGW についてくる windres.exe があれば、別に [rsrc.exe](https://github.com/akavel/rsrc) が無くてもよかった

# 手順

nyagos.ico というアイコンがある場合は、次のようなテキストファイルを用意する

```nyagosico.rc
#include <winver.h>

GOPHER ICON nyagos.ico
```
(GOPHER はリソース名なので、別に何でもよい)

windres.exe で、.syso ファイルを作成する。

```
windres --output-format=coff -o nyagosico.syso nyagosico.rc
```

この .syso ファイルが、*.go ファイルと同じフォルダーにあれば、`go build` を実行するだけで、.syso ファイルは自動で取り込まれる。

# 補足

* 実は[前回](http://qiita.com/zetamatta/items/4aa36661336a56b95e06)も一度この方法を試してはいたのだが、`go build ファイル名` という形で実行していたのと、拡張子が .syso でなければいけないという2点にひっかかっていたのであった。
* rc ファイルをちゃんと書けば、たぶん exe ファイルに(プロパティーで確認できる)バージョンナンバーを組み込むこととかもできるだろう。

