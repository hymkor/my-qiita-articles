---
title: [go] cgoは使いたくないけど、C.GoBytes() は使いたいみたいなー [追記あり]
tags: Go:1.3
author: zetamatta
slide: false
---
cgo は結構万能だけど、これを使うと、Windows の場合、ビルドの際、MinGW gcc が必要になりそうな雰囲気である。これでは多くの pull request が期待できないッ。でも、C.GoBytes() とか、C.GoStringN() を使いたいッ。この気持ちッ、天に届け！

# <del>でけた！</del> <ins>できてへん</ins>

```go
var msvcrt = syscall.NewLazyDLL("msvcrt")
var memcpy = msvcrt.NewProc("memcpy")

func CGoBytes(p, length uintptr) []byte {
	buffer := make([]byte, length)
	memcpy.Call(uintptr(unsafe.Pointer(&buffer[0])), p, length)
	return buffer
}

func CGoStringN(p, length uintptr) string {
	return string(CGoBytes(p, length))
}
```

# これではうまくいかない

* length が 0 の時に panic を起こす
    * これについては `if length <= 0 { return []byte{} }` すれば Ok
* Ctrl-C ハンドラがうまく動作しなくなる
    * `import "C"` を入れるだけで動作するようになる、なぜ？

ということで、C.GoBytes は不要にはなりましたが、import "C" だけは必要という、あまり意味のない状態で、現在に至ります。わからんなー

# import "C" せずに Ctrl-C ハンドラを正常動作させる回避策

`kernel32.dll` の `SetConsoleCtrlHandler` を使わないようにしたら、Ctrl-C がキャッチできるようになった。理由はよくわからないが、シグナル関係が Windows 版 Go ではまだきちんと実装されていないことが関係しているのだろうか…

## 関係URL
* [@zetamatta 気になったので調べてみたんですが、conio.OnClose が呼ばれると ENABLE_PROCESSED_INPUT が無効になってしまうみたいです](https://twitter.com/hattya/status/533981437255380992)
* [@zetamatta nyagos.go の conio.OnClose をコメントアウト、conio/import_c.go を削除して試してみると Ctrl+C で入力がキャンセルされました](https://twitter.com/hattya/status/533984715804389377)
* [zetamatta / nyagos / Save history at each input and remove cgo](https://github.com/zetamatta/nyagos/commit/087e2104bfe8c338e1ff66b01cf8b4d2bdce20e8) 

