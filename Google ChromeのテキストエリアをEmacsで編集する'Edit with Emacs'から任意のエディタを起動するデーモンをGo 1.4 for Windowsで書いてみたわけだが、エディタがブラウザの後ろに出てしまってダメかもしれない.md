---
title: Google ChromeのテキストエリアをEmacsで編集する'Edit with Emacs'から任意のエディタを起動するデーモンをGo 1.4 for Windowsで書いてみたわけだが、エディタがブラウザの後ろに出てしまってダメかもしれない
tags: Go:1.4
author: zetamatta
slide: false
---
どういう経緯か忘れたのですが

- [Google Chrome のテキストエリアを外部エディタで編集する Edit with Emacs](http://sfujiwara.hatenablog.com/entry/20100129/1264743217)

という記事を拝見しました。概要としては

- Google Chrome は Firefox とは違い、外部プロセスを起動することは禁止されている
- そこで Chrome Extension の [Edit with Emacs](https://chrome.google.com/webstore/detail/edit-with-emacs/ljobjlafonikaiipfkggjbhkghgicgoh) は、Emacs のサービスを 127.0.0.1:9292 に立ち上げて、XMLHttpRequest を使って、テキストエリアの内容を Emacs と交換している。
- 上記の記事では、Emacs のサーバーのかわりに Perl でサービスを立ち上げて、任意のエディターを Chrome で使えるようにしている

という感じです。記事の方法だと Perl 一式が必要だし、Linux でしか動作が検証されていないようでした（別に Windows で動かないとは書いてないが）。が、記事のおかげで原理はだいたい分かったので、勉強がてら Go 1.4 for Windows で「だいたい同じもの」を作ってみました。

- [Any Editor's Server for chrome-extension: 'Edit with Emacs'](https://github.com/zetamatta/editsrv)

```editsrv.go
package main

import (
	"bufio"
	"fmt"
	"html"
	"io"
	"io/ioutil"
	"net/http"
	"os"
	"os/exec"
	"strings"
)

func HasHtml(headers map[string][]string) bool {
	xurl, ok := headers["X-Url"]
	if !ok || len(xurl) <= 0 {
		return false
	}
	return strings.HasPrefix(xurl[0], "https://twitter.com")
}

func typeHeaders(h map[string][]string, w io.Writer) {
	for key, vals := range h {
		for _, val := range vals {
			fmt.Fprintf(w, "%s: %s\n", key, val)
		}
	}
}

func html2text(out io.Writer, in io.Reader) {
	scanner := bufio.NewScanner(in)
	for scanner.Scan() {
		line := scanner.Text()
		fmt.Fprintln(os.Stderr, line)
		line = strings.TrimSpace(line)
		line = strings.Replace(line, "<div><br></div>", "\n", -1)
		line = strings.Replace(line, "<div>", "", -1)
		line = strings.Replace(line, "</div>", "\n", -1)
		line = html.UnescapeString(line)
		io.WriteString(out, line)
	}
}

func text2html(out io.Writer, in io.Reader) {
	scanner := bufio.NewScanner(in)
	for scanner.Scan() {
		line := scanner.Text()
		line = strings.TrimSpace(line)
		line = html.EscapeString(line)
		if line == "" {
			line = "<br>"
		}
		fmt.Fprintf(out, "<div>%s</div>", line)
	}
}

func handler(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(os.Stderr, "From: %s\n", req.RemoteAddr)
	typeHeaders(req.Header, os.Stderr)
	tmpfd, tmpfdErr := ioutil.TempFile("", "editsrv")
	if tmpfdErr != nil {
		fmt.Fprintln(os.Stderr, tmpfdErr)
		return
	}
	io.WriteString(tmpfd, "\xEF\xBB\xBF")
	tmpName := tmpfd.Name()
	defer os.Remove(tmpName)

	hasHtml := HasHtml(req.Header)
	if hasHtml {
		html2text(tmpfd, req.Body)
	} else {
		io.CopyN(tmpfd, req.Body, req.ContentLength)
	}
	tmpfd.Close()

	var editorName string
	if len(os.Args) >= 2 {
		editorName = os.Args[1]
	} else {
		editorName = "notepad.exe"
	}
	editorArgs := make([]string, 0, len(os.Args))
	for i := 2; i < len(os.Args); i++ {
		editorArgs = append(editorArgs, os.Args[i])
	}
	editorArgs = append(editorArgs, tmpName)
	cmd1 := exec.Command(editorName, editorArgs...)
	fmt.Fprintf(os.Stderr, "Call %s ", editorName)
	for _, arg1 := range editorArgs {
		fmt.Fprintf(os.Stderr, " %s", arg1)
	}
	fmt.Fprint(os.Stderr, "\n")
	if err := cmd1.Run(); err != nil {
		fmt.Fprintln(os.Stderr, err)
		return
	}
	tmpfd, tmpfdErr = os.Open(tmpName)
	if tmpfdErr != nil {
		fmt.Fprintln(os.Stderr, tmpfdErr)
		tmpfd.Close()
		return
	}
	fmt.Fprintf(os.Stderr, "Send '%s' to Chrome\n", tmpName)
	if hasHtml {
		text2html(w, tmpfd)
	} else {
		_, copyErr2 := io.Copy(w, tmpfd)
		if copyErr2 != nil {
			fmt.Fprintln(os.Stderr, copyErr2)
		}
	}
	tmpfd.Close()
	fmt.Fprintln(os.Stderr, "Done")
}

func main() {
	fmt.Println("Any Editor Server for chrome-extension 'Edit with Emacs'")
	http.HandleFunc("/edit", handler)
	http.ListenAndServe(":9292", nil)
}
```

最初、よー分からんで何故か TCP のレベルからやろうとして挫折したのですが、net/http で書き直したところ、比較的簡単に出来ました。「`editsrv.exe gvim.exe`」を走らせておくと、テキストエリアの右下に出る `edit` のマークをクリックした時に gvim.exe が起動するようになります。(なお、UTF8 対応エディターである必要があります)

ただ、twitter の本家サイトについては、テキストエリアの内容が何故か HTML で飛んでくるので、このサイトだけは特別扱いして、HTML ⇔ プレーンテキストの変換を相互にやってます。**やりすぎかな**。

普通、この手のものは「思いついたのはいいが、なかなかうまいこと作れない」ものなんですが、自分にしては、あっさりできて驚きでした。ま、多分、似たようなものの Go 版は先人がとっくの昔に作ってそうではありますが…まぁ、手段が目的みたいなもんですから、そこんところは目をつぶってくださいませ。

**しかし…起動したエディターがブラウザの後ろに隠れてしまうのはどうしたもんかな…**


