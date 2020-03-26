---
title: 【Go言語】Excelize なにもわからない【Excel帳票】
tags: Go Excel
author: zetamatta
slide: false
---
最近、Go言語の中華系Excel操作ライブラリ [excelize](https://github.com/360EntSecGroup-Skylar/excelize) を使うことが多い。いろいろ便利で重宝しているが、難しい点もあったので、報告したいと思う。

なお、動作を保証するものではないので、ご利用は OWN YOUR RISK で。また、事実と異なる点などあったら、コメントなどでご指摘いただきたい。

使用雑感
=======
* シート・セルごとにいちいちオブジェクトのクラスは存在しない。ほとんどのメソッドは excelize.File型からいきなり生えており、シート名、セル名をパラメータで指定する<br />**（話が早くてよい）**
* セル名は "A1" といった文字列で指定するメソッドが多い<br />**（分かりやすいけど、めんどくさい）**
    * 一応、セル名のアルファベット文字部分を数値から作成する補助関数は存在する
* 新規作成・既存ファイルの読み書きともに問題は見受けられない
* 画像の読み出し・貼り付けも Ok 。ただし、読み出しの際はセル名の指定が必要<br />**（詳しくは[Go言語で Excel へのスクショ貼りを自動化しよう！](https://qiita.com/zetamatta/items/8c953f05771dcd10b6ed)を参照のこと）**
* グラフが売りらしい（が、自分は未検証）
* セルの修飾はいろいろ出来るが、**これが結構難しい。**

セルの修飾
=========

（罫線・背景色など…）は可能だが、やや癖がある。修飾内容（スタイルという）は JSON 文字列で表現する（以下 GoDoc の例に改行だけ追加したもの）

```go
{"border":[{"type":"left","color":"0000FF","style":3},
{"type":"top","color":"00FF00","style":4},
{"type":"bottom","color":"FFFF00","style":5},
{"type":"right","color":"FF0000","style":6},
{"type":"diagonalDown","color":"A020F0","style":7},
{"type":"diagonalUp","color":"A020F0","style":8}]}
```

この文字列を 

```go
func (*excelize.File)NewStyle(style string)(int,error)
```

というメソッドで スタイルの ID に変換した上で

```go
func (*excelize.File)SetCellStyle(sheet, hcell, vcell string, styleID int)
```

というメソッドでセルに設定する。

* IDは Excel 帳票ごとにローカルなものなので、あらかじめ既存のセルについて調べておいて、プログラム中でハードコーディングなどすると、panic が発生することがある**（おこした）**
* スタイルの読み出しは可能だが（`GetCellStyle`）、IDしか分からない。そして、**IDから JSON 文字列への逆変換方法はない**。
* **スタイルは、元々のセルのスタイルを完全に上書きする**。つまり、背景色のあるセルに罫線を足そうとすると、背景色が消えてしまう。

スタイルの上書きを避けるためには、一つの JSON 文字列の中でフォント・罫線・背景など最終状態を全部表現してしまわないといけない。だが、文字列なので誤記するとエラーが実行時に起こってしまうし、スタイルを多数用意しなければいけないときに同じような内容を何回も書かないといけなくなってしまう。

さすがにこれは辛いので、自分はJSON文字列とIDを生成するための構造体などを用意した（https://github.com/zetamatta/go-excelize-assist ）。以下の例では構造体の Compile メソッドでスタイルIDを得ている。

```go
import (
    // 中略
    "github.com/360EntSecGroup-Skylar/excelize"
    "github.com/zetamatta/go-excelize-assist/xstyle"
)
// 中略
xlsx := excelize.NewFile()

font := &xstyle.Font{
        Size:   9,
        Family: "ＭＳ ゴシック",
}
bold := &xstyle.Font{
        Size:   9,
        Family: "ＭＳ ゴシック",
        Bold:   true,
}
tate := &xstyle.Alignment{
        Rotation: 90,
}
fill := &xstyle.Fill{
        Type:    "pattern",
        Color:   []string{"#E0EBF5"},
        Pattern: 1,
}
border := xstyle.NewBorder(xstyle.JISSEN, xstyle.JISSEN, xstyle.JISSEN, xstyle.JISSEN)

bodyStyle, _ := (&xstyle.Style{Font: font, Borders: border}).Compile(xlsx)
headerStyle, _ := (&xstyle.Style{Font: font, Alignment: tate, Borders: border}).Compile(xlsx)
headerBoldStyle, _ := (&xstyle.Style{Font: bold, Alignment: tate, Borders: border}).Compile(xlsx)
fillBodyStyle, _ := (&xstyle.Style{Font: font, Fill: fill, Borders: border}).Compile(xlsx)
fillHeaderStyle, _ := (&xstyle.Style{Font: font, Alignment: tate, Fill: fill, Borders: border}).Compile(xlsx)
fillHeaderBoldStyle, _ := (&xstyle.Style{Font: bold, Alignment: tate, Fill: fill, Borders: border}).Compile(xlsx)
```

今後、excelize の方で、このあたりも改良されて欲しいところだ。ただ、なんとなくポリシーあってわざとやってそうにも見えるので、なかなか安易には変えてもらえないかもしれない。

（なんか、ちょうど、スタイルの上書きに関する質問とかも出てるみたい → [The cell color how to set? Can give a simple example? · Issue #254 · 360EntSecGroup-Skylar/excelize](https://github.com/360EntSecGroup-Skylar/excelize/issues/254)）

シートの表示オプションなど
=========
次のようにすれば、ワークシートのデフォルトの罫線の Off が出来た。

```go
err := xlsxfile.SetSheetViewOptions(sheet, -1, excelize.ShowGridLines(false))
```

このシートのオプションを設定する関数の仕様は下記のとおりだが、オプションの指定の仕方が面白い。

```go
func (f *File) SetSheetViewOptions(name string, viewIndex int, opts ...SheetViewOption) error
```

オプション名である `ShowGridLines` は定数や関数ではなく、SheetViewOptionインターフェイスを実装する「型名」として定義されている。なかなか使えそうなテクニックだ。

----

以上、また、何か発見があったら、追記したい。

