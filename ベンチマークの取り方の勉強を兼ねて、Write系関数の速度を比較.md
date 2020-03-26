---
title: ベンチマークの取り方の勉強を兼ねて、Write系関数の速度を比較
tags: Go
author: zetamatta
slide: false
---
```io_test.go
package bench

import (
	"fmt"
	"io"
	"io/ioutil"
	"os"
	"testing"
)

func BenchmarkWriteString(b *testing.B){
	for i := 0 ; i < b.N ; i++ {
		io.WriteString(ioutil.Discard,"hogehoge")
	}
}

func BenchmarkFmtFprint(b *testing.B){
	for i := 0 ; i < b.N ; i++ {
		fmt.Fprint(ioutil.Discard,"hogehoge")
	}
}

func BenchmarkWriteWithStr2Byte(b *testing.B){
	for i := 0 ; i < b.N ; i++ {
		ioutil.Discard.Write( []byte("hogehoge") )
	}
}

func BenchmarkWrite(b *testing.B){
	hogehoge := []byte("hogehoge")
	for i := 0 ; i < b.N ; i++ {
		ioutil.Discard.Write( hogehoge )
	}
}

func BenchmarkWriteStringReal(b *testing.B){
	fd,_ := os.Create("nul")
	fd.Close()
	for i := 0 ; i < b.N ; i++ {
		io.WriteString(fd,"hogehoge")
	}
}
```

* ソースファイル名は、テスト系の機能なので「`_test.go`」とつけておくこと

```run.cmd
go test -bench .
```

```
goos: windows
goarch: amd64
pkg: github.com/zetamatta/experimental/bench
BenchmarkWriteString-4          100000000               18.8 ns/op
BenchmarkFmtFprint-4            20000000                74.7 ns/op
BenchmarkWriteWithStr2Byte-4    50000000                28.9 ns/op
BenchmarkWrite-4                500000000                3.61 ns/op
BenchmarkWriteStringReal-4      10000000               124 ns/op
PASS
ok      github.com/zetamatta/experimental/bench 8.550s
```

結論

* 速い ：Write([]byte) ＞ io.WriteString ＞ fmt.Fprint：遅い（これは当然の結果）
* ただし、下手に Write([]byte("...") とキャストを入れるよりは io.WriteString("...") の方がマシ

