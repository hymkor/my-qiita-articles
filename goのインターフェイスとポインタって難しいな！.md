---
title: goのインターフェイスとポインタって難しいな！
tags: Go:1.3
author: zetamatta
slide: false
---
```hoge.go
package main

import "io"
import "os"
import "fmt"

func end(fd *io.Closer){
	fd.Close()
}

func main(){
	fd,_ := os.Create("hoge.txt")
	fmt.Fprintln(fd,"ahahaha")
	end(fd)
}
```

コンパイルすると

```text
[C:gosrc]
$ go run hoge.go
# command-line-arguments
.\hoge.go:8: fd.Close undefined (type *io.Closer has no field or method Close)
.\hoge.go:14: cannot use fd (type *os.File) as type *io.Closer in argument to end:
        *io.Closer is pointer to interface, not interface
[C:gosrc]
$
```

なぜだ、os.File には、Close というメソッドがあるじゃないか。io.Closer の条件になぜ合致しないんだ。コンパイルをあきらめるなよ、もっと熱くなれよ。

ためしに、関数 end の引数の型をポインタではなく、値にすると

```hoge.go
package main

import "io"
import "os"
import "fmt"

func end(fd io.Closer){
	fd.Close()
}

func main(){
	fd,_ := os.Create("hoge.txt")
	fmt.Fprintln(fd,"ahahaha")
	end(fd)
}
```

なんか、うまく動いたっぽい。

```
[C:gosrc]
$ go run hoge.go
[C:gosrc]
$ type hoge.txt
ahahaha
```

エラーメッセージをよく読むと、<b>「*io.Closer はインターフェイスへのポインタであって、インターフェイスそのものちゃうで。<del>あほちゃうか？</del>」</b>と書いてあるなぁ。そうか、インターフェイスのポインタはインターフェイスとみなしてくれないのか。難しいな！

# 追記

[[教えて]Go言語:なぜインターフェイスはポインタにできない?](http://qiita.com/suin/items/68ed7020d21dca047a73)という記事があった。なるほど、インターフェイスのポインタには、インターフェイスとしての機能はないか。

