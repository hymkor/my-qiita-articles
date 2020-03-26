---
title: goによるテキストファイル操作テンプレ（自分用）
tags: Go
author: zetamatta
slide: false
---
git が何かとエスケープシーケンスを吐くんだけど、more とか nkf を通すと、Windows ではゴミになっちゃうので、それを除くフィルターを作った。

ノウハウというよりは、自分用のテンプレとして。
(Python とかでもいいんだけど、インストールしてない時もあるので)

```ansistrip.go
package main

import "bufio"
import "io"
import "os"
import "regexp"
import "fmt"

var ansiStrip = regexp.MustCompile("\x1B[^a-zA-Z]*[A-Za-z]")

func cat1(r io.Reader) {
	scanner := bufio.NewScanner(r)
	for scanner.Scan() {
		fmt.Println(ansiStrip.ReplaceAllString(scanner.Text(), ""))
	}
}

func main() {
	for _, arg1 := range os.Args[1:] {
		r, err := os.Open(arg1)
		if err != nil {
			fmt.Fprintln(os.Stderr, err.Error())
			return
		}
		cat1(r)
		r.Close()
	}
	if len(os.Args) <= 1 {
		cat1(os.Stdin)
	}
}
```

* 2014/07/22 コード差し替え
   * WriteString() を2回実行して、1行を出力していたのを、fmt.Fprintln を使うようにした
   * defer r.Close() を使うと、プログラム終了までファイルを握りっぱなしになるので、即 Close() するようにした。
   * 正規表現オブジェクトを素直にグローバルにした。
   * https://github.com/zetamatta/ansistrip にソースを登録

