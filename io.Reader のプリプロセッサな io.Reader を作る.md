---
title: io.Reader のプリプロセッサな io.Reader を作る
tags: Go
author: zetamatta
slide: false
---
* （アドベントカレンダー、穴が開いたようなので、埋めさせていただきます）
* （12/24晩、ベンチマーク結果などを追記しました）

任意の io.Reader を受け取って、それを加工した結果を、別の io.Reader として読み取れるようにしたい時、どうするのがベストな方法だろうか。

具体的には、当初「文字コード(ShiftJIS or UTF8)を UTF8 に変換するフィルターを作る」といったことをしたかったのだが、文字コード変換まで含めるとコードの本当に説明したい箇所がぼやけるので、本文書では簡単に「行番号を付加する」といった例を用いて「**io.Reader to io.Reader なフィルターの出来るだけお手軽な作り方**」を検討した結果を報告したいと思う。

io.Pipe で作る
==============

加工処理を別の goroutine で行い、io.Pipe の Writer の方に加工結果を出力する。ユーザは io.Pipe の Reader 側を使えばよい。

```lnum1.go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func lnum(src io.Reader) io.Reader {
	in, out := io.Pipe()
	lnum := 0
	go func() {
		br := bufio.NewReader(src)
		for {
			line, err := br.ReadBytes('\n')
			if err != nil {
				out.CloseWithError(err)
				return
			}
			lnum++
			fmt.Fprintf(out, "%d: %s", lnum, line)
		}
	}()
	return in
}

func main() {
	sc := bufio.NewScanner(lnum(os.Stdin))
	for sc.Scan() {
		fmt.Println(sc.Text())
	}
	if err := sc.Err(); err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}
}
```

「`go run lnum1.go < ファイル名`」のようにして使う。この方法はお手軽で、標準ライブラリだけで完結する。ただ、goroutine を別途立ち上げるので（大したことはないだろうが）余分なリソースを使う点、また途中で読むのを止めた時に goroutine が残ってしまわないよう対策が必要な点などが課題だ。

"tidwall/transform" を使う
==========================

こういうフィルター処理に最適なライブラリを探していたところ、[よいもの](https://github.com/tidwall/transform)が見つかった。

```lnum2.go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"

	"github.com/tidwall/transform"
)

func lnum(src io.Reader) io.Reader {
	br := bufio.NewReader(src)
	lnum := 0
	return transform.NewTransformer(func() ([]byte, error) {
		line, err := br.ReadString('\n')
		if err != nil {
			return nil, err
		}
		lnum++
		return []byte(fmt.Sprintf("%d: %s", lnum, line)), nil
	})
}

func main() {
	sc := bufio.NewScanner(lnum(os.Stdin))
	for sc.Scan() {
		fmt.Println(sc.Text())
	}
	if err := sc.Err(); err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}
}
```

特に問題ないが、強いて言うならば標準ライブラリで完結しないくらい？（それは問題ちゃうやろう）

"golang.org/x/text/transform"
=============================

こちらは準・標準的なライブラリで、

```golang
type Transformer interface {
    Transform(dst, src []byte, atEOF bool) (nDst, nSrc int, err error)
    Reset()
}
```

というインターフェイスで型を実装すれば、

```golang
transform.NewReader(元Reader, 実装したTransformerのインスタンス)
```

という形式で、変換を実装する io.Reader を作成してくれる。

が、**これ Transform メソッドを実装するのが非常に面倒くさい。src や dst のサイズが任意ではなく、呼び出し元から決められているせいだ**。一回、実装してみたが、dst があふれそうな時、別のバッファに残しておいて次回に処理するとか、なかなか大変だった（やり方が悪いのかもしれない）。

ただ、メモリアローケーション回数が少ないはずなので、速度的には有利で、うまく実装できれば非常に実行効率がよいフィルターが作れると思われる。

なんかよい実装例ないものか（誰か、わかりやすい日本語の解説記事書いてよ）

以上

追記
====

`"tidwall/transform"` と似たようなものを作ろうとしたのですが、ベンチを取ると、やはり`"tidwall/transform"` にはかないませんでした。

```
goos: windows
goarch: amd64
pkg: github.com/zetamatta/go-texts/preprocessor
Benchmark_filter-4        	   10000	    109332 ns/op	   18220 B/op	     321 allocs/op
Benchmark_transformer-4   	   10000	    108897 ns/op	   15627 B/op	     319 allocs/op
Benchmark_iopipe-4        	   10000	    230557 ns/op	   15874 B/op	     323 allocs/op
PASS
ok  	github.com/zetamatta/go-texts/preprocessor	4.733s
```

ソースは https://github.com/zetamatta/go-texts/blob/master/preprocessor/preprocess_test.go を御覧ください

* `Benchmark_filter-4` … 自作品
* `Benchmark_transformer-4` … `"tidwall/transform"`
* `Benchmark_iopipe-4` … io.Pipe と goroutine によるもの

io.Pipe と goroutine では仕組みが大掛かりなせいか、２倍近くコストがかかっています。自作のものは bytes.Buffer を２つ交互に使ってバッファリングしているのですが、あと一歩及びません。

`"tidwall/transform"`のソースを見てみると、[]bytes を使って溢れたデータを管理しているのですが、領域を開放せずに長さだけゼロにする `(bytes.Buffer)Reset` 的な操作を

```go
r.buf = r.buf[:0] // reset the read buffer, keeping it's capacity
```

とやっていました。同操作を`[]byte` でやるのにどうするんだと思っていたのですが、なるほどなぁ…です。



