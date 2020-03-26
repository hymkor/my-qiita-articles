---
title: nyagosスクリプト解説 - 逆クォートによるコマンド出力展開編(backquote.lua)
tags: nyagos
author: zetamatta
slide: false
---
標準で搭載されている（=`nyagos.d\`以下に置かれている）ので、nyagos.exe 組込み機能かと思われているかもしれないが、nyagos の「逆クォートによるコマンド出力の展開機能」はスクリプトとフックで実現されている。

nyagos.filter はコマンドライン全部を加工することができるフィルター関数を代入することができるフックである。[nyagos.d\backquote.lua](https://github.com/zetamatta/nyagos/blob/master/nyagos.d/backquote.lua) では、これを利用して逆クォート展開を実現している。

```Lua:backquote.lua
local orgfilter = nyagos.filter
nyagos.filter = function(cmdline)
    if orgfilter then
        local cmdline_ = orgfilter(cmdline)
        if cmdline_ then
            cmdline = cmdline_
        end
    end
```

nyagos.filter には一つの関数しか代入できないので、最初に行うのは、まず、元々入っていた nyagos.filter の関数のバックアップを取っておき、新関数側でコマンドラインの加工をする前に元々代入されていた関数を先に呼び出してやるということになる。nyagos.filter は nil を返す時は、加工をキャンセルすることを示すので、そのあたりも処理してやる必要がある。**（が、今考えると、わざわざ if 文をつらつら書かなくとも「`cmdline = (orgfilter and orgfilger(cmdline)) or cmdline`」とだけ書けばよかったのだったー）**

```lua
    return string.gsub(cmdline,'`([^`]*)`',function(m)
        local r = nyagos.eval(m)
        if not r then
            return false
        end
        r = nyagos.atou(r)
        r = string.gsub(r,'[|&<>!]',function(m)
            return string.format('%%u+%04X%%',string.byte(m,1,1))
        end)
        return string.gsub(r,'%s+$','')
    end)
end
```

次に本番の処理を行う。

nyagos.eval は「かなりおなじみ」の関数だが、一つだけ注意する点がある。それは戻り値が現在のコードページの文字列であるということだ。これは「**コマンドの出力がテキストであるとは限らないので、下手に UTF8 に変換できない**」という **nyagos.eval の仕様**なので、ご理解いただきたい。

しかしながら、nyagos のコマンドライン自体は UTF8 なので、さすがに現在のコードページ文字列をそのまま処理するわけにはいかない。ということで、UTF8 とコードページ文字列を相互変換する関数 nyagos.atou と nyagos.utoa で変換してから、逆クォート部分と置き換えている**（つまり、逆クォートの中で、現在のコードページで表現できない文字が出力されるとゴニョゴニョ）**

あと、backquote.lua では、逆クォートの他に %u+NNNN% というユニコードリテラルの展開もやっている**（とソースを見て思い出した）**。

以上、だいたいお分かりいただけただろうか






