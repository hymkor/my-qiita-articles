---
title: 本当に Windows10 のコマンドプロンプトはエスケープシーケンスをサポートしたのか？
tags: Go
author: zetamatta
slide: false
---
したっぽい。実験してみよう

```windows10.go
package main

import (
	"fmt"
	"os"
	"syscall"
	"unsafe"
)

var kernel32 = syscall.NewLazyDLL("kernel32")

const STD_INPUT_HANDLE = uintptr(1) + ^uintptr(10)
const STD_OUTPUT_HANDLE = uintptr(1) + ^uintptr(11)
const STD_ERROR_HANDLE = uintptr(1) + ^uintptr(12)
const ENABLE_VIRTUAL_TERMINAL_PROCESSING uintptr = 0x0004

var procGetStdHandle = kernel32.NewProc("GetStdHandle")
var procGetConsoleMode = kernel32.NewProc("GetConsoleMode")
var procSetConsoleMode = kernel32.NewProc("SetConsoleMode")

func Main() error {
	var mode uintptr
	console, _, _ := procGetStdHandle.Call(STD_OUTPUT_HANDLE)

	rc, _, err := procGetConsoleMode.Call(console, uintptr(unsafe.Pointer(&mode)))
	if rc == 0 {
		return err
	}
	defer procSetConsoleMode.Call(console, mode)

	rc, _, err = procSetConsoleMode.Call(console, mode|ENABLE_VIRTUAL_TERMINAL_PROCESSING)
	if rc == 0 {
		return err
	}
	println("\x1B[32;1mAHAHA\x1B[37;1m")
	return nil
}

func main() {
	if err := Main(); err != nil {
		fmt.Fprintln(os.Stderr, err.Error())
		os.Exit(1)
	} else {
		os.Exit(0)
	}
}
```

![image.png](https://qiita-image-store.s3.amazonaws.com/0/29454/d43e61d5-88b5-a9c1-7dcd-a33c9961159c.png)

SetConsoleMode で、`ENABLE_VIRTUAL_TERMINAL_PROCESSING` (0x0004) というビットを立てなくてはいけないようです。

くわしい仕様はこちら：

* [Console Virtual Terminal Sequences (Windows)](https://msdn.microsoft.com/en-us/library/windows/desktop/mt638032(v=vs.85).aspx)

Go でやる分には [go-colorable](https://github.com/mattn/go-colorable) があるので、あんまりすぐ使う必要性はありませんが、いつが Windows 7/8 がサポート切れになった時、使う日が来るかもしれません。

