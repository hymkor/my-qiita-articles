---
title: Go言語で、よう分からん error の正式な型名を確認する
tags: Go
author: zetamatta
slide: false
---
エラーメッセージがダサいので差し替えようと思ったが、その型名がそもそもよう分からん時は、reflect パッケージで調べられる。

```go:foo.go
package main

import (
	"os"
	"reflect"
)

func TypeName(obj interface{}) string {
	return reflect.TypeOf(obj).String()
}

func main() {
	_, err := os.Stat("NOT_FOUND_FILE")
	println(err.Error())
	println(TypeName(err))
	println("os.IsNotExist=" , os.IsNotExist(err))
	println("os.IsExist=" , os.IsExist(err))
}
```

```shell-session
$ go run foo.go
CreateFile NOT_FOUND_FILE: The system cannot find the file specified.
*os.PathError
os.IsNotExist= true
os.IsExist= false
```

os.Stat でファイルが存在しない時のエラーは、`os.PathError` へのポインタだった。
なお、普通は `os.IsNotExist(エラーオブジェクト)` を使う模様。

reflect パッケージ、邪悪ですばらしいですね！

