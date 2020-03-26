---
title: Windows で vipw っぽく、システムのデフォルト%PATH%を変更する
tags: Windows:7 VBScript:5.8
author: zetamatta
slide: false
---
という VBScript を書いてみました(Windows7 向け)

1. 起動すると、UAC のダイアログが表示されます。
2. 「はい(Y)」とすると、メモ帳で %PATH% の内容が開かれます
（この時点で、セミコロン「`;`」は改行に置換されています）。
4. 修正してセーブすると（改行をセミコロン「`;`」に置き換えて）システム環境変数のデフォルト値へ変更内容を書き込みます。

お使いの PC の設定を変更するスクリプトですので、 **ご利用は計画的に**。

```chpath.vbs
if WScript.Arguments.Count <= 0 then
    Set shellApp = CreateObject("Shell.Application")
    shellApp.ShellExecute "wscript.exe","""" & WScript.ScriptFullName & """ uac" , "" , "runas"
    Set shellApp = Nothing
    WScript.Quit
end if
Set wshShell=CreateObject("WScript.Shell")
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set sysEnv=wshShell.Environment
Path = sysEnv.Item("Path")
TempName = objFSO.GetTempName()

Set objWriter = objFSO.CreateTextFile(TempName,True,False)
objWriter.WriteLine( Replace(Path,";",vbcrlf) )
objWriter.Close()
Set objWriter = Nothing

Set objExec = wshShell.Exec("notepad " & TempName)
Do While objExec.Status = 0
    WScript.Sleep 100
Loop
output = objExec.StdOut.ReadAll
if len(output) > 0 then
    WScript.Echo output
end if
output = objExec.Stderr.ReadAll
if len(output) > 0 then
    WScript.Echo output
end if

Set objReader = objFSO.OpenTextFile(TempName,1)
Path = Replace(objReader.ReadAll,vbcrlf,";")
if Right(Path,1) = ";" then
    Path = Left(path,len(path)-1)
end if
objReader.Close()
Set objReader = nothing

rem WScript.Echo Path
objFSO.DeleteFile(TempName)
sysEnv.Item("Path") = Path
Set objFSO = Nothing
```

# 参考にしたページ

* [Server World - VBScript を管理者権限で実行](http://www.server-world.info/query?os=Other&p=vbs&f=1)

