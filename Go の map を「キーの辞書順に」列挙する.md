---
title: Go の map を「キーの辞書順に」列挙する
tags: Go
author: zetamatta
slide: false
---
map を**キーの辞書順に列挙**する場合、普通は次のように、一旦キーをスライスに書きだして、それをソートして range を使うのが定番とされている。

```foo.go
package main

import (
	"sort"
)

func main() {
	map1 := map[string]int{
		"z": 1, "a": 10, "c": 4, "d": 5,
	}
        
	// キーをソートする
	slice1 := make([]string, len(map1))
	index := 0
	for key := range map1 {
		slice1[index] = key
		index++
	}
	sort.Strings(slice1)

	// 列挙
	for _, key := range slice1 {
		println(key, map1[key])
	}
}
```

```
$ go run foo.go
a 10
c 4
d 5
z 1
```

**これはめんどくさい！**

せめてソート部分を共通関数化できれば…。でも、Generics がないので、map の型ごとにいちいち実装しなくてはいけない。それでは共通化の意味がない。

**こんな時は、やはり reflect ですか？先生！**

```bar.go
package main

import (
	"reflect"
	"sort"
)

func SortedKeys(mapInt interface{}) []string {
	values := reflect.ValueOf(mapInt).MapKeys()
	result := make([]string, len(values))
	for i, value1 := range values {
		result[i] = value1.String()
	}
	sort.Strings(result)
	return result
}

func main() {
	map1 := map[string]int{
		"z": 1, "a": 10, "c": 4, "d": 5,
	}

	for _, key := range SortedKeys(map1) {
		println(key, map1[key])
	}

	map2 := map[string]string{
		"bb": "hoge", "aa": "uuuuu", "cc": "o00000",
	}

	for _, key := range SortedKeys(map2) {
		println(key, map2[key])
	}
}
```

```
$ go run bar.go
a 10
c 4
d 5
z 1
aa uuuuu
bb hoge
cc o00000
```

map の「キー」の型は string 固定だけど、「値」の型については何でも Ok になりました（※ただし、速いとは言っていない）



