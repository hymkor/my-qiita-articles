---
title: go 1.3 と Windows における日本語ファイル名 
tags: Go:1.3
author: zetamatta
slide: false
---
何も考えずに書いてみる

```readdirs.go
package main

import "fmt"
import "os"

func main(){
	fd , err := os.Open(".")
	if err != nil {
		fmt.Fprintln(os.Stderr,err)
		return
	}
	nodes , err := fd.Readdir(-1)
	if err != nil {
		fmt.Fprintln(os.Stderr,err)
		return
	}
	fd.Close()
	for _,n := range nodes {
		fmt.Println(n.Name())
	}
}
```

えーと、「☑」は SJIS にはなかったよね？

![スクリーンショット 2014-07-06 22.55.39.png](e5015490-8e8f-4184-a284-33499ca8311b.png)

goだと、特に迷うことはない。Python も同様に早く Windows の Unicode API に対応してほしいです。(それとも、Python 3 とかだと対応してるのかな？)

