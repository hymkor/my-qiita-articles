---
title: Enum の値から、シンボル文字列を得る
tags: VB.Net:2010
author: zetamatta
slide: false
---
型が変わってなかったら ToString だけでよかったんかー。別に GetName とか使わんでよかったんや…（涙目）

```vb
Module Module1
    Enum NumberEnum
        ZERO
        ONE
        TWO
        THREE
        FOUR
        FUOR = FOUR
        FIVE
        SIX
        SEVEN
        EIGHT
        NINE
        TEN
    End Enum

    Sub Main()
        For i As NumberEnum = NumberEnum.ZERO To NumberEnum.TEN
            Console.WriteLine(String.Format("{0}={1}", i.ToString(), [Enum].GetName(GetType(NumberEnum), i)))
        Next
        Console.ReadLine()
    End Sub
End Module
```

```vb
ZERO=ZERO
ONE=ONE
TWO=TWO
THREE=THREE
FUOR=FUOR
FIVE=FIVE
SIX=SIX
SEVEN=SEVEN
EIGHT=EIGHT
NINE=NINE
TEN=TEN
```

本当は 4 が来た時に、FOUR が来るか、FUOR になるか、確認するだけのはずあったんや…

# 追記

どうも、FOUR がくるか、FUOR が来るかは、不定（未定義？）のような雰囲気。[Enum].GetNames メソッドで、全要素の文字列を取得するのが安全のような雰囲気だ。

[Enum.ToString メソッド (String)](http://msdn.microsoft.com/ja-jp/library/a0h36syw(v=vs.110).aspx)

> 複数の列挙型のメンバーの基になる値が同じであり、その基になる値に基づいて列挙型メンバーの名前の文字列形式を取得しようとする場合、メソッドから返される名前についてコードで一切の前提を排除する必要があります。 たとえば、次の列挙型では Shade.Gray と Shade.Grey という 2 つのメンバーが定義されていますが、これらの基になる値は同じです。

> 次のメソッド呼び出しは、基になる値が 1 である Shade 列挙体のメンバーの名前を取得しようとします。 メソッドで "Gray" または "Grey" を返すことはできますが、特定の文字列が返されることを想定したコーディングは避けてください。

# 追記２

* [VB.NET Enumの使い方(まとめ)](http://programmers.high-way.info/vb/enum.html)

おれには、[Enum].TryParse(Of 型)(文字列,変数) があれば十分やったんやー

