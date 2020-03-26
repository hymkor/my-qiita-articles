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

![スクリーンショット 2014-07-06 22.55.39.png](https://qiita-image-store.s3.amazonaws.com/0/29454/08d35e4c-3269-7481-ae1d-4f4073715755.png)

goだと、特に迷うことはない。Python も同様に早く Windows の Unicode API に対応してほしいです。(それとも、Python 3 とかだと対応してるのかな？)

