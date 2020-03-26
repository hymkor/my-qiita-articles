---
title: えぇ、このソース GoSub なんてあるのかい！？
tags: VB:6 VB.Net:2010
author: zetamatta
slide: false
---
VB6 の GoSub は、VB.Net に移植する際はクロージャーで書き直すのが一番てっとり早いんではなかろうか

```VB6
Sub Foo()
    '変数宣言もろもろ
    '処理もろもろ
    GoSug Hoge

Hoge:


    Return
End Sub
```

```vbnet
Sub Foo()
   '変数宣言もろもろ
   Dim Hoge As Action = 
       Sub()

       End Sub
   '処理もろもろ
   Hoge.Invoke()

End Sub
```

ただ、これを入れたら入れたで、

```
'On Error GoTo' ステートメントを含むメソッドに、ラムダを生成する式を含めることはできません。
```

というエラーになるので、On Error GoTo を Try～Catch に書き直す作業が発生するが、これは仕方がないよね。

