---
title: nyagosスクリプト解説 - CMD.EXEで化けさせず、nyagosの中だけプロンプトをカラー化
tags: nyagos
author: zetamatta
slide: false
---
（コメント欄にて、nyagos 4.1 向け修正点を追記しています）

そのままエスケープシーケンスを %PROMPT% に代入しちゃうと、CMD.EXE を実行した時に色がつかず、エスケープシーケンスコードが「文字」として出てしまう。そこで、nyagos.prompt フックを使おう！

```Lua:.nyagos
-- Simple Prompt for CMD.EXE
nyagos.env.prompt='$L'.. nyagos.env.computername .. ':$P$G$_$$$s'
```

まずは CMD.EXE 向けに普通に %PROMPT% を設定する。自分は VM を含む、いろんな環境で .nyagos を共通化させており(`mklink /S %USERPROFILE%\.nyagos %USERPROFILE%\OneDrive\etc\.nyagos` してる )、PC 名(%COMPUTERNAME%) もプロンプトに含んでいるが、**ここは好みで変えていただきたい**。

```Lua:.nyagos
-- Coloring Prompt for NYAGOS.exe
do
    local prompter=nyagos.prompt
    nyagos.prompt = function(this)
        return prompter('$e[36;40;1m'..this..'$e[37;1m')
    end
end
```

- 一番外側の do～loop は単にローカル変数 prompter の有効範囲を狭めているだけなので、気にしなくてよろしい。
- nyagos.prompt はプロンプトを出す関数が格納されている。これを横取りする。
- nyagos.prompt は第一引数に %PROMPT% の内容が渡される。別に nyagos.env.prompt から得ても良いが、nyagos.prompt を入れ子などした場合、ここに改変された後のプロンプトのフォーマット文字列が入ってくる可能性があるので、全てのプロンプト書き換え関数を活かすのであれば、引数の方を使った方がよろしい
- nyagos.prompt を実行するのは nyagos.exe だけなので、ここで前後にエスケープシーケンスを挟み、prompter に退避した**旧のnyagos.prompt**を呼び出してプロンプトを表示させる。

### 余談だが

自分はプロンプトに改行コードを入れてる。nyagos はコマンドラインが長い場合、折り返さず、横にスクロールさせる。これは倍角文字列が行の一番最後で表示されて行を跨いだ場合、コンソールが勝手に半角スペースを挟んでしまい、文字がどこにあるかをプログラムが把握できなくなってしまうのを避けるためにとった処置だ。だが、このせいで編集の空間が小さくなってしまう。改行を入れてるのは、できるだけ編集の空間を広げようという苦肉の策なのだ。(´・ω・｀)

