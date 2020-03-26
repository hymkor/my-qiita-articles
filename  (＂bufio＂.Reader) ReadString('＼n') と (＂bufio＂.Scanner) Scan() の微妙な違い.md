---
title:  ("bufio".Reader) ReadString('\n') と ("bufio".Scanner) Scan() の微妙な違い
tags: Go
author: zetamatta
slide: false
---
明確な違い
===

* `("bufio".Reader) ReadString('\n')`
    * 得られる行文字列（戻り値）は改行コードを含む
* `("bufio".Scanner) Scan()`
    * 得られる行文字列（`Text()`の戻り値）は改行コードを含まない

改行コードのない最終行の扱い
=====================

最後の行が `abcd`**[EOF]** というテキストの場合

* `("bufio".Reader) ReadString('\n')`
    * {`"abcd",io.EOF`}という戻り値となる
* `("bufio".Scanner) Scan()`
    * `Text()` は "abcd" という戻り値を返し、「次回」の `Scan()` が false になる

Scanner の方は定番であるところの、

```go
for sc.Scan() {
   line := sc.Text()
   fmt.Println(line)
}
```

という書き方で問題ないが、Reader の方は

```go
for{
    line,err := r.ReadString('\n')
    if err != nil {
        break
    }
    fmt.Print(line)
}
```

だと、最後の行が抜けてしまう。

```go
for{
    line,err := r.ReadString('\n')
    fmt.Print(line)
    if err != nil {
        break
    }
}
```

と書かないといけない（厳密には io.EOF 以外のエラーの時の処理を分けるべきだが…）。こう書くと「どんなケースでも常に Scanner を用いる」のが妥当なように思われるが、Scanner は最終行の改行の有無がわからないので、入力したデータをできるだけ変えずに出力するようなフィルター処理には向かない。

