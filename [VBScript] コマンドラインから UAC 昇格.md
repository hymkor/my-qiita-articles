---
title: [VBScript] コマンドラインから UAC 昇格
tags: VBScript:5
author: zetamatta
slide: false
---
`cscript please.vbs プログラム名` とすると、管理者権限でそのプログラムを起動します。プログラム名を省略すると、CMD.EXEが起動します。

直前に投稿した「[Windows で vipw っぽく、システムのデフォルト%PATH%を変更する](http://qiita.com/zetamatta/items/2919f0a794e6373eb964)」の副産物です。

```VBScript:please.vbs
Option Explicit
Dim program
If WScript.Arguments.Count <= 0 Then
    program = "CMD.EXE"
Else
    program = WScript.Arguments(0)
End If
Dim argv : argv = ""
If WScript.Arguments.Count >= 1 then
    Dim i
    For i=1 To WScript.Arguments.Count-1
        argv = argv & " " & WScript.Arguments(i)
    Next
End If
Dim shellApp : Set shellApp = CreateObject("Shell.Application")
shellApp.ShellExecute program,Trim(argv), "" , "runas"
Set shellApp = Nothing
```

