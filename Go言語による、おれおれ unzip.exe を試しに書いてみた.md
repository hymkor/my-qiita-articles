---
title: Go言語による、おれおれ unzip.exe を試しに書いてみた
tags: Go:1.3
author: zetamatta
slide: false
---
Linux 上で圧縮したZIPファイルを、Windowsで展開した時、UTF-8 ファイル名が化けてしまうので、Go言語でさくっと「おれおれ unzip」を書いてみました。いやぁ、きょうびの言語は、さくっと１時間程度で書けるくらい、ライブラリが充実しているもんですね。まぁ、明らかに車輪の再発明ですが、勉強にはいいものです。

ちなみに、巷には UTF-8 → SJIS でファイル名を変換するアーカイバもあると思うんですが、そうすると３か国語の文字が混ざるとアウトなので、UTF-8 → UTF-16 経由で変換する方式でないとまずいんですが、Go言語はそういうのをライブラリで吸収してくれるので、楽でいいですね！（Javaとか.NETとかでも大丈夫でしょうけども）

```ungop.go
package main

import "archive/zip"
import "fmt"
import "io"
import "os"

func main() {
	if len(os.Args) < 2 {
		return
	}
	reader, readerErr := os.Open(os.Args[1])
	if readerErr != nil {
		fmt.Fprintf(os.Stderr, "%s: %s\n", os.Args[1], readerErr.Error())
		return
	}
	defer reader.Close()
	finfo, finfoErr := reader.Stat()
	if finfoErr != nil {
		fmt.Fprintf(os.Stderr, "%s: %s\n", os.Args[1], finfoErr.Error())
		return
	}
	zipReader, zipReaderErr := zip.NewReader(reader, finfo.Size())
	if zipReaderErr != nil {
		fmt.Fprintf(os.Stderr, "%s: %s\n", os.Args[1], zipReaderErr.Error())
		return
	}
	for _, f := range zipReader.File {
		zipFileReader, zipFileReaderErr := f.Open()
		if zipFileReaderErr != nil {
			fmt.Fprintf(os.Stderr, "%s: %s: %s\n",
				os.Args[1],
				f.Name,
				zipFileReaderErr.Error())
			continue
		}
		unzipWriter, unzipWriterErr := os.Create(f.Name)
		if unzipWriterErr != nil {
			fmt.Fprintf(os.Stderr, "%s: %s: %s\n",
				os.Args[1],
				f.Name,
				unzipWriterErr.Error())
		} else {
			_, err := io.Copy(unzipWriter, zipFileReader)
			if err != nil {
				fmt.Fprintf(os.Stderr, "%s: %s: %s\n",
					os.Args[1],
					f.Name,
					err.Error())
			} else {
				fmt.Println(f.Name)
			}
			unzipWriter.Close()
		}
		zipFileReader.Close()
	}
}
```

あんまり真剣にテストしてないので、不具合あったら、ごめんなさい。つーか、多分、逆にSJISのファイル名だったりすると、即アウトでしょう、きっと。

