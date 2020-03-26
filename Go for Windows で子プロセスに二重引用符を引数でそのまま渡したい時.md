---
title: Go for Windows で子プロセスに二重引用符を引数でそのまま渡したい時
tags: Go Windows
author: zetamatta
slide: false
---
問題の症状
=========

親プロセスのソース：

```exec1.go
package main

import (
	"os"
	"os/exec"
)

func main() {
	c := exec.Command("foo", `"<BAR>"`)
	c.Stdout = os.Stdout
	c.Stderr = os.Stderr
	c.Stdin = os.Stdin
	c.Run()
}
```

子プロセスのソース（バッチファイル）：

```foo.cmd
@echo %0 %*
@exit /b 0
```

実行すると

```console
$ go run exec1.go
foo \"<BAR>\"
```

二重引用符の前にバックスラッシュがついてしまう。

解決方法
=======

バックスラッシュが付かないようにするには、Go言語標準の`exec.Cmd.Args[]`を Windows 形式の引数形式に展開する処理をパスして、自前で直接指定すればよい。

```exec2.go
package main

import (
	"os"
	"os/exec"
	"syscall"
)

func main() {
	c := exec.Command("foo")
	c.SysProcAttr = &syscall.SysProcAttr{CmdLine: `foo "<BAR>"`}
	c.Stdout = os.Stdout
	c.Stderr = os.Stderr
	c.Stdin = os.Stdin
	c.Run()
}
```

```console
$ go run exec2.go
foo "<BAR>"
```

なぜ、こんなことをする必要があるのか
================================

FIND.EXE や CMD.EXE は、引数につけられた二重引用符の有無で、その引数がどういうものか判別しています。このため、Goのプログラムから FIND.EXE が意図どおりに呼べないという問題が発生していました。

* [dir | find "<DIR>" · Issue #218 · zetamatta/nyagos](https://github.com/zetamatta/nyagos/issues/218)

こういう仕様って、どこに書いてあったの？
====================================

[ここ](https://github.com/golang/go/blob/master/src/syscall/exec_windows.go#L267)

