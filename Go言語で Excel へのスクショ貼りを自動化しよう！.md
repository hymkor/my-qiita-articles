---
title: Go言語で Excel へのスクショ貼りを自動化しよう！
tags: Go Excel
author: zetamatta
slide: false
---
Go で Excel ファイルを作成するのに使えるライブラリとしては

* [go-ole/go-ole](https://github.com/go-ole/go-ole)
    * Excel自身をOLE経由で動かすので互換性の懸念はない。xlsx だけでなく、xls でもいける。だけど **遅い**。要Excel。結構めんどうくさい（ミスるとExcelプロセスがゾンビになって残る）
* [tealeg/xlsx](https://github.com/tealeg/xlsx)
     * **定番**。読み取り・新規作成はバッチリだが、既存のxlsxファイルを加工する際に画像などが飛んでしまうと言われている。また、メモリ消費量が多い
* [loadoff/excl](https://github.com/loadoff/excl)
     * xlsx と違って、**既存Excelファイルへの加工に心配がない**。メモリ消費量が抑えられている。開発者日本人。Qiita に[紹介記事](https://qiita.com/tebakane/items/2f2ed2558357c274c478)あり

などが知られているが、画像などが貼れるものということで、今回、[tealeg/xlsxのトップ画面](https://github.com/tealeg/xlsx) で「`You should probably also checkout`」とか言って紹介されていた [360EntSecGroup-Skylar/excelize](https://github.com/360EntSecGroup-Skylar/excelize)を使ってみた。

```shot.go
package main

import (
	"fmt"
	"os"
	"path/filepath"

	_ "image/gif"
	_ "image/jpeg"
	_ "image/png"

	"github.com/360EntSecGroup-Skylar/excelize"
)

func mains() error {
	xlsx := excelize.NewFile()

	for _, arg := range os.Args[1:] {
		name := filepath.Base(arg)
		xlsx.NewSheet(name)
		if err := xlsx.AddPicture(name, "A1", arg, ""); err != nil {
			return err
		}
	}
	xlsx.DeleteSheet("Sheet1")

	return xlsx.SaveAs("./Book1.xlsx")
}

func main() {
	if err := mains(); err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}
}
```

`go run shot.go ./2018-08-19_01.15.57.png`とかすると、`2018-08-19_01.15.57.png` というシートに `2018-08-19_01.15.57.png` が貼り付けられた Book1.xlsx というファイルが生成される。

なお、以下の import が抜けると、画像が貼り付けが行われない上に、それがエラーとして通知もされないので注意。

```
    _ "image/gif"
    _ "image/jpeg"
    _ "image/png"
```

以上

