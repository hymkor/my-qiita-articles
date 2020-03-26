---
title: 2次元配列をReDimする
tags: VB.Net:2010
author: zetamatta
slide: false
---
うん、馬鹿馬鹿しいね。でも、VB6 からコンバートしてたら、そんなソースができちゃったんだ

で、関数作った。

```vbnet
    Public Sub Redim2(Of t)(ByRef array(,) As t, w As Integer, h As Integer)
        If array Is Nothing Then
            ReDim array(w, h)
        ElseIf array.GetUpperBound(1) < h Then
            ReDim Preserve array(array.GetUpperBound(0), h)
        ElseIf array.GetUpperBound(0) < w Then
            Dim newarray(,) As t = New t(w, array.GetUpperBound(1)) {}
            For i As Integer = 0 To array.GetUpperBound(0)
                For j As Integer = 0 To array.GetUpperBound(1)
                    newarray(i, j) = array(i, j)
                Next
            Next
            array = newarray
        End If
    End Sub
```

