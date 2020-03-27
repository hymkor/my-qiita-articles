---
title: DataGridView でセルの入力チェック、自動補正、自動確定/却下など
tags: .NET:4.0
author: hymkor
slide: false
---
DataGridView とは、便利なスプレッドシートっぽいコントロールでいろんなことができて便利だが、小難しいことをやる時には、 小難しいことをやらないといけなくて、そろその脳がオーバーフローしつつある。

いちいち過去のソースを見るのも面倒になってきたので、ここに覚えがきを書いておく。

# セルが不正な値の時に却下する

CellValidating をハンドルして、e.FormattedValue に入っている値をチェックするロジックを入れる。

不正な値の時は「CancelEdit()」メソッドを実行すると、変更前の値に戻る。 あと「e.Cancel = True」も必要(理由は後述)

# セルの内容を自動補完する

「1」を入力した時に自動で「001」に補完したい時は、DataGridView の CellParsing をハンドルする。 「e.Value」に入力値が入っているので、それを補正した値で上書きして、 「e.ParsingApplied = True」する。

# セルの編集中(鉛筆印表示)のまま、DataGridView以外のコントロールで操作された時、編集を確定or却下させる。

Enterキーを押下したり、DataGridView の別のセルに移動したら編集は確定されるのだが、 そうではなく、同じフォームの「メニュー」項目をクリックしたり、 クローズボタンを押した場合は確定されず、編集中状態のまま、イベントが開始されてしまう。

こういう場合、編集中状態を解除するには 2種類方法があって、 (DataGridView).EndEdit か、(フォームインスタンス).Validate のいずれかを呼ぶ。

前者を呼ぶと編集は強制確定されるのだが、CellValidating とか呼ばれないので、 おかしな値がそのまま入ってしまうこともある。よってお勧めしない。

後者の場合、フォーム中のコントロール全て(つまり DataGridView含む)に対して登録操作が試みられる上に、 ちゃんと DataGridView では CellValidating が呼ばれるので、不正な値もちゃんとはじいてもらえる。

で、ここで値をはじいた場合、新に行おうとした操作もキャンセルすべきだろう (編集前の不本意な値で操作を続行されると不都合な結果になる)。 ここで生きているのが、先程の「CellValidating での e.Cancel = True」で、これをやっておくと、 (フォームインスタンス).Validate() の戻り値が False になってくれる。 だから、このコードは定番は次のようなものになる。

```vbnet 
If Not Me.Validate() Then
    Return
End If
```

# 編集を破棄して終了していいですか？

フォームの FormClosing をハンドルする。 MsgBox とかでダイアログを出して、クローズをキャンセルさせる場合は「e.Cancel = True」。

あと、この時も、編集しかけの DataGridView を確定・却下させるべきだが、 却下した後、そのまま Return すると、そのままクローズされてしまうので、 先程の定番コードは、ここだけ

```vbnet 
If Not Me.Validate() Then
    e.Cancel = True
    Return
End If
```

になる。

以上、http://nyaos.org/d/index.cgi?p=%282013.06.01%29+0943#p1 より転記
