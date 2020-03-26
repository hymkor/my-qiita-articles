---
title: VMごとにいちいち環境変数HOMEを GUI で設定するのが面倒なので
tags: VBScript
author: zetamatta
slide: false
---
環境変数のほとんどは nyaos の設定ファイル(%HOME%\_nya) に定義しているが、基準となる HOME だけはマイコンピュータのプロパティー経由で設定してやらなくてはいけないのは面倒。そこでHOME を設定する VBScript を用意した。

```setHome.vbs
Set argv=WScript.Arguments

Set shell=CreateObject("WScript.Shell")
Set userEnv=shell.Environment("User")
if argv.Count < 1 then
    WScript.Echo("Current HOME=" & userEnv.Item("HOME"))
Else
    If Len(argv(0)) > 0 then
        userEnv.Item("HOME") = argv(0)
    Else
        userEnv.Remove("HOME")
    End If
End If
```

* `cscript setHome.vbs` で現在の内容を表示。
* `cscript setHome.vbs 新しい値` で環境変数の初期値を更新します。

----

* 2014.08.23(修正)""を引数として渡せば、%HOME%を削除できるようにした。

