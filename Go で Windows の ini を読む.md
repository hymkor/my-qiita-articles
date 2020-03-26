---
title: Go で Windows の ini を読む
tags: Go:1.3 Windows:7
author: zetamatta
slide: false
---
老害乙です、こんにちは！ 退屈しのぎに、件のコードを書いてみました。

```getprivateprofilestring.go
package kernel32ini

import (
	"syscall"
	"unsafe"
)

var kernel32 = syscall.NewLazyDLL("kernel32")
var getPrivateProfileString = kernel32.NewProc("GetPrivateProfileStringW")

func GetPrivateProfileString(path, section, keyname, defaultValue string) (string, error) {
	var buffer [2048]uint16

	sec16, secErr := syscall.UTF16PtrFromString(section)
	if secErr != nil {
		return "", secErr
	}
	key16, keyErr := syscall.UTF16PtrFromString(keyname)
	if keyErr != nil {
		return "", keyErr
	}
	default16, defaultErr := syscall.UTF16PtrFromString(defaultValue)
	if defaultErr != nil {
		return "", defaultErr
	}
	path16, pathErr := syscall.UTF16PtrFromString(path)
	if pathErr != nil {
		return "", pathErr
	}
	result, _, err := getPrivateProfileString.Call(
		uintptr(unsafe.Pointer(sec16)),
		uintptr(unsafe.Pointer(key16)),
		uintptr(unsafe.Pointer(default16)),
		uintptr(unsafe.Pointer(&buffer[0])),
		uintptr(len(buffer)),
		uintptr(unsafe.Pointer(path16)))

	if result <= 0 {
		return "", err
	}
	return syscall.UTF16ToString(buffer[0:result]), nil
}
```

テストコード：

```getprivateprofilestring_test.go
package kernel32ini

import (
	"fmt"
	"os"
	"path"
	"testing"
)

const (
	INIPATH = "getprivateprofilestring.ini"
	SECTION = "section1"
	KEY     = "key1"
	DEFAULT = ""
)

func TestGetPrivateProfileString(t *testing.T) {
	wd, err := os.Getwd()
	if err != nil {
		fmt.Println(err.Error())
		t.Fail()
		return
	}
	iniPath := path.Join(wd, INIPATH)
	value, err := GetPrivateProfileString(iniPath, SECTION, KEY, DEFAULT)
	if err != nil {
		fmt.Println(err.Error())
		t.Fail()
		return
	}
	fmt.Printf("%s[%s]%s=%s\n", iniPath, SECTION, KEY, value)
}
```

P.S. 良い子は [TOML](http://qiita.com/futoase/items/fd697a708fcbcee104de) 使った方がいいと思います！

