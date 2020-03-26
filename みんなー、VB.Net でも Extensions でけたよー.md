---
title: みんなー、VB.Net でも Extensions でけたよー
tags: VB.Net:2010
author: zetamatta
slide: false
---
* [[C#]ちょっとしたストレスを解消してくれるExtensions](http://qiita.com/shinoyu/items/7dec4bbd0282ae6755e6)

VB.Net でもやりてー！ということでググってみると、できるみたい。

* [拡張メソッド (Visual Basic)](http://msdn.microsoft.com/ja-jp/library/bb384936(v=vs.100).aspx)

書いてみた

```vbnet
Imports System.Runtime.CompilerServices

Public Module StringExntensions
    <Extension()>
    Public Function IsNullOrEmpty(ByVal this As String) As Boolean
        Return String.IsNullOrEmpty(this)
    End Function

    <Extension()>
    Public Function IsNullOrWhiteSpace(ByVal this As String) As Boolean
        Return String.IsNullOrWhiteSpace(this)
    End Function

    <Extension()>
    Public Function Format(ByVal this As String, ByVal ParamArray args() As Object) As String
        Return String.Format(this, args)
    End Function
End Module
```

```vbnet
Module Module1
    Sub Main()
        Console.WriteLine("Test1=" & "".IsNullOrEmpty())
        Console.WriteLine("Test2=" & "".IsNullOrWhiteSpace())
        Console.WriteLine("{0} & {1}".Format(2, 3))
        Console.ReadLine()
    End Sub
End Module
```

```
Test1=True
Test2=True
2 & 3
```

やればできるんだよ、あきらめるなよ、もっと熱くなれよ！



