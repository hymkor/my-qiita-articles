---
title: コマンドライン（バッチファイル）から明示的に UAC 昇格する一番短い方法
tags: PowerShell
author: zetamatta
slide: false
---
* 2018.01.19 追記：もっと短い方法がありました
    * [管理者権限でbatを実行したい時にやッた事](https://qiita.com/resolver/items/7187bd6ee8016ee5c741)

-------

今まで [VB.Net](https://github.com/zetamatta/wouldyou) で書いたり、[バッチファイル＋JScript](https://gist.github.com/zetamatta/6148dc1aecdc78d83add) を書いたり、いろいろ考えてきたけど、PowerShell ならワンライナーで出来たー

```
powershell "$obj=New-Object -Com Shell.Application ; $obj.ShellExecute('cmd.exe','/k dir','','runas')"
```

なお、これは COM を使っているんだけど、本来は .NET 方面のオブジェクトを使った方がいいんでしょうね

