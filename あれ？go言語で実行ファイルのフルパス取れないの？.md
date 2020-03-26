---
title: あれ？go言語で実行ファイルのフルパス取れないの？
tags: Go:1.3,1.8
author: zetamatta
slide: false
---
# Go 1.8 向け追記

[os.Executable](https://golang.org/pkg/os/#Executable)という標準関数で取得できるようになっているようです。

# 元記事


```args0.go
package main

import "fmt"
import "os"

func main(){
        fmt.Printf("os.Args[0]=\"%s\"\n",os.Args[0])
}
```

```text
[C:gosrc]
$ go run args0.go
os.Args[0]="C:\Users\Hayama\AppData\Local\Temp\go-build331085211\command-line-arguments\_obj\exe\args0.exe"
```

うん、`go run`だと問題ない。では、コンパイルしてみよう。

```text
[C:gosrc]
$ go build args0.go
[C:gosrc]
$ args0.exe
os.Args[0]="args0.exe"
[C:gosrc]
$ cd ..
[C:\Opt]
$ gosrc\args0.exe
os.Args[0]="gosrc\args0.exe"
```

あれぇぇぇ？

# (追記) 仕方ないので、モジュール書いた

2回もループ回したり、べたべたやけど

```exename.go
package exename

//#include <windows.h>
import "C"

import "bytes"
import "unicode/utf16"

func Query() string {
	var pathW [C.MAX_PATH]C.WCHAR
	C.GetModuleFileNameW(nil, &pathW[0], C.MAX_PATH)

	var path16 [C.MAX_PATH]uint16
	for i := 0; pathW[i] != 0; i++ {
		path16[i] = (uint16)(pathW[i])
	}

	pathRune := utf16.Decode(path16[:])
	var buffer bytes.Buffer
	for _, ch := range pathRune {
		if ch == 0 {
			break
		}
		buffer.WriteRune(ch)
	}
	return buffer.String()
}
```

