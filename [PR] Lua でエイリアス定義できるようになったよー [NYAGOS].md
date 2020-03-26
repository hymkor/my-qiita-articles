---
title: [PR] Lua でエイリアス定義できるようになったよー [NYAGOS]
tags: Go:1.3 Lua:5.2 nyagos:20140809
author: zetamatta
slide: false
---
[NYAGOS なら Unicode ファイル名にも自信が持てます](http://qiita.com/zetamatta/items/c66197b37244e7b0804c)

> Lua で（カレントディレクトリや環境変数を操作できるような）内蔵コマンドも「まだ」作れない。

**[NYAGOS](http://www.nyaos.org/index.cgi?p=NYAGOS)でも、できるようになったよー**

[NYAOS-3](http://www.nyaos.org/index.cgi?p=NYAOS+3000) と違って、エイリアスとLua関数を等価的に扱えるぞ！

---

通常は NYAGOS ではエイリアスは `nyagos.alias("名前","コード")` と定義できるんじゃが。
起動時に読み込まれる nyagos.lua にて

```lua:nyagos.lua
local function split(equation)
    local pos=string.find(equation,"=",1,true)
    if pos then
        local left=string.sub(equation,1,pos-1)
        local right=string.sub(equation,pos+1)
        return left,right,pos
    else
        return nil,nil,nil
    end
end

function alias(equation)
    if type(equation) == 'table' then
        for left,right in pairs(equation) do
            nyagos.alias(left,right)
        end
    else
        local left,right,pos = split(equation)
        if right then
            nyagos.alias(left,right)
        end
    end
end
```

と定義しているので

```lua:nyagos.lua
alias{
    assoc='%COMSPEC% /c assoc $*',
    attrib='%COMSPEC% /c attrib $*',
    copy='%COMSPEC% /c copy $*',
    del='%COMSPEC% /c del $*',
    dir='%COMSPEC% /c dir $*',
    ['for']='%COMSPEC% /c for $*',
    md='%COMSPEC% /c md $*',
    mkdir='%COMSPEC% /c mkdir $*',
    mklink='%COMSPEC% /c mklink $*',
    move='%COMSPEC% /c move $*',
    open='%COMSPEC% /c for %I in ($*) do @start "%I"',
    rd='%COMSPEC% /c rd $*',
    ren='%COMSPEC% /c ren $*',
    rename='%COMSPEC% /c rename $*',
    rmdir='%COMSPEC% /c rmdir $*',
    start='%COMSPEC% /c start $*',
    ['type']='%COMSPEC% /c type $*',
    ls='ls -oF $*',
}
```

と書く事ができる。

※ Lua では、テーブル一つだけを引数に取る関数コール「`f( {a=b} )`」を「`f{a=b}`」と記述することが出来る。 

んでもって、今日書いたばかりのコードによってじゃ…

```lua:.nyagos
alias{
    hogehoge = function(...) 
        local args={...}
        for i=1,#args do
            print(string.format("[%d]=%s",i,args[i]))
        end
    end 
}
```

と書く事ができるんじゃ。(※2015.04.09追記参照)

```text
<C:\Opt>
$ alias hogehoge
hogehoge=<<Lua-function>>
<C:\Opt>
$ hogehoge 1 2 3 4 5
[1]=hogehoge
[2]=1
[3]=2
[4]=3
[5]=4
[6]=5
<C:\Opt>
```

すごいじゃろ、すごいじゃろ…すごいと言ってくれぇ…

----

ちなみに [NYAOS-3](http://www.nyaos.org/index.cgi?p=NYAOS+3000) でも同様のことはできるが、[NYAOS-3](http://www.nyaos.org/index.cgi?p=NYAOS+3000) ではエイリアスは nyaos.alias、lua関数は nyaos.command と別に分けていた。

----

しかし、この記事、Goタグついてるけど、Goネタが全然書いてない。えー、一応申し上げますと、NYAGOSはGo製のWindows向けコマンドラインシェルです。


**※(2015.04.09) 追記、現行バージョンでは次のように書く必要があります。**

```lua
alias{
    hogehoge = function(args) 
        for i=1,#args do
            print(string.format("[%d]=%s",i,args[i]))
        end
    end 
}
```

