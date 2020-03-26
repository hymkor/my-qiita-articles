---
title: 二重引用符などで面倒なことにならない cmd.exe の呼び方
tags: Go Windows
author: zetamatta
slide: false
---
環境変数を使おう

```system.go
package dos

import (
	"os"
	"os/exec"
)

func System(cmdline string) error {
	const CMDVAR = "CMDVAR"

	orgcmdarg := os.Getenv(CMDVAR)
	defer os.Setenv(CMDVAR, orgcmdarg)

	os.Setenv(CMDVAR, cmdline)

	cmd1 := exec.Command("cmd.exe", "/c", "%"+CMDVAR+"%")
	cmd1.Stdout = os.Stdout
	cmd1.Stderr = os.Stderr
	cmd1.Stdin = os.Stdin
	return cmd1.Run()
}
```

```system_test.go
package dos

import (
	"testing"
)

func TestSystem(t *testing.T) {
	err := System("echo \"ahaha ihihi\"")
	if err != nil {
		t.Fatalf("%s\n", err.Error())
	}
}
```

```
$ go test -v
=== RUN   TestSystem
"ahaha ihihi"
```

※テストがおかしいのは分かっているので、つっこまないでくださいｗ

