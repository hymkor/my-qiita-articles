---
title: go/Windows の filepath.Join はダメっぽい
tags: Go:1.3
author: zetamatta
slide: false
---
```pathcombine.go
package main

import "fmt"
import "os"
import "path/filepath"

func main(){
	fmt.Println(filepath.Join(os.Args[1:]...))
}
```

```text
$ go run pathcombine.go c:\opt \ahaha b:\ihihi
c:\opt\ahaha\b:\ihihi
```

にょおおおお

# Pythonだと

期待どおりの動作をする

```pathcombine.py
import os.path
import sys

print os.path.join(*sys.argv[1:])
```

```text
$ ipy pathcombine.py c:\Opt \ahaha b:\ihihii
b:\ihihii
```

# （追記）VB.Netでも

期待どおりの動きをする

```pathcombine.vb
 Module Module1
    Sub Main()
        Dim args As String() = System.Environment.GetCommandLineArgs()
        If args.Count >= 3 Then
            Dim args2 As String() = New String(args.Count - 2) {}
            Array.Copy(args, 1, args2, 0, args.Count - 1)

            Console.WriteLine(System.IO.Path.Combine(args2))
            Console.ReadLine()
        End If
    End Sub
End Module
```

コマンドライン引数

```text
c:\opt \ahaha b:\ihihi
```

結果

```text
b:\ihihi
```

# （追記）ということで

自分はとりあえず、簡単なラッパーをかまして逃げた。([元ソース](https://github.com/zetamatta/nyagos/blob/master/dos/dos.go))

```dos.go
package dos
// 中略
var rxDriveOnly = regexp.MustCompile("^[a-zA-Z]:$")
var rxRoot = regexp.MustCompile("^([a-zA-Z]:)?[\\/]")

func Join(paths ...string) string {
	start := 0
	for i, path := range paths {
		if rxDriveOnly.MatchString(path) {
			paths[i] = path + "."
		} else if rxRoot.MatchString(path) {
			start = i
		}
	}
	if start > 0 {
		paths = paths[start:]
	}
	return filepath.Join(paths...)
}
```

これ、具体的にどこで使っているかというと、[go製のコマンドラインシェル NYAGOS](https://github.com/zetamatta/nyagos) の内蔵 [ls](https://github.com/zetamatta/nyagos/blob/master/ls/ls.go) でございます。

