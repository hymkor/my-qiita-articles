---
title: goで作るexeファイルにアイコンを埋め込んでみる (Windows)
tags: Go:1.3
author: zetamatta
slide: false
---
# 参考サイト

* [[ANN] rsrc - Tool for embedding Windows executable manifests in Go programs.](https://groups.google.com/forum/#!topic/Golang-Nuts/shRlhV3Fe2E)
* https://github.com/akavel/rsrc

# やってみた

```
go get github.com/akavel/rsrc
cd %GOPATH%/src/github.com/akavel/rsrc
go build
copy rsrc.exe %HOME%\bin\.

cd (自分のソースディレクトリ)
rsrc -ico nyagos.ico -o nyagos.syso
```

作成された「nyagos.syso」は、組み込まれる go ソースのあるのと同じフォルダに置いておけば「go build」で自動的に組み込まれるとある。

# 注意すべきこと

`go build` だと組み込まれるが、`go build nyagos.go` など、具体的なソース名を引数に指定してしまうとアウトらしい。

# 余談

アイコンは [DotWork](http://www5a.biglobe.ne.jp/~suuta/soft.html#dotwork)というツールで作りました。

