---
title: a[:0] と append の秘密
tags: Go
author: zetamatta
slide: false
---
（http://zetamatta.hatenablog.com/entry/2019/02/05/161003 より転載）

[io.Reader のプリプロセッサな io.Reader を作る](https://qiita.com/zetamatta/items/9312375f90f7b5bee85f)の中で、["tidwall/transform"](https://github.com/tidwall/transform)で使われていると紹介したテク：`a = a[:0]` は領域のサイズをリセットするが、`a = make([]T,0,cap(a))` と違って、使っていたメモリブロックを再利用するため、allocation 回数を削減できるというものだった。

だが、旧`a` の領域が他で使われていないかを気にせず、無頓着に使うと append でおかしいことになる。たとえば

```go
a := []string{ "a","b","c" }
b := a[:0]
b = append(b,"1")

fmt.Printf("%+v\n",a)
```

とすると a の内容が [1 b c] となる。これは append が領域を上書きで使用するためだ。

append 側で他所で使われているかチェックしてくれたらよさそうな話ではあるが、おそらく物理的に無理だろう。というのも参照カウンタ方式ではなく、マーク＆スイープ方式の Garbage Collector を使っている場合、参照されている場所が自分を含めて２個以上あるか、調べようがないのだ。（GCの中身のコードならばできなくもないだろうが…）

となると、常に他で使われているという前提で append を実装しないと、上のような状況は避けられないわけだ。そういう append ちょっと作ってみよう。

```go
package main

import (
	"fmt"
)

func appendStr(a []string, b string) []string {
	r := make([]string, len(a)+1)
	copy(r,a)
	r[len(a)] = b
	return r
}

func main() {
	a := []string{}
	for i := 0; i < 10; i++ {
		a = appendStr(a, fmt.Sprintf("%d", i))
	}
	fmt.Printf("%+v\n", a)
}
```

こんな実行効率わるそうなの、みんな使いたいと思うだろうか？結局、みんなオリジナル append を使うことになるだろう？（でも、スクリプト言語では効率が悪くても平気でやってそうではある）。

-----

こういった append の挙動は一見奇異なものに見え「Go言語は糞」と罵られる一因ではある。だが、C言語を知る者は append は realloc と対応する関数だから、領域拡張前の a が正常に使える保証はなく、使うべきではないと直感的に分かっている人が多い。

人によって、Go言語の評価が「サイコー」or「糞」と正反対になるのは、そういうあまり言及されない前提知識や慣れ（訓練されたC言語プログラマはエラーチェックコードが多くてもまったく気にしない等）の有無が大きいのではないだろうか。

