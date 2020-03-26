---
title: nyagosスクリプト解説 - ezoe.lua「コマンドではない。」
tags: nyagos
author: zetamatta
slide: false
---
catalog.d\ に格納されている[お遊びスクリプトの一つ](https://github.com/zetamatta/nyagos/blob/master/catalog.d/ezoe.lua)。コマンドが無い時にフック `nyagos.on_command_not_found` に関数が登録されていると、その関数が呼ばれる機能を利用している（**というか、このために作った**）。

```ezoe.lua
nyagos.on_command_not_found = function(args)
    nyagos.writerr(args[0]..": コマンドではない。\n")
    return true
end
```

また、cd で存在しないディレクトリに移動した時にも「ディレクトリではない。」と出すようにした。これは cd をエイリアス定義することで実現している。

```lua
local cd = nyagos.alias.cd
```

**cd は実は結構多重にエイリアス定義されている**。なので、先に定義されているエイリアスをローカル変数に退避しておく。

```lua
nyagos.alias.cd = function(args)
    local success=true
    for i=1,#args do
        local dir=args[i]
        if dir:sub(1,1) ~= "-" and
            not dir:match("%.[lL][nN][kK]$") and
            not dir:match("^[a-zA-Z]:[/\\]$") and
            not dir:match("^[/\\]$") and
            dir ~= ".." and
            dir ~= "." then

            local stat = nyagos.stat(dir)
            if not stat or not stat.isdir  then
                nyagos.writerr(dir..": ディレクトリではない。\n")
                success = false
            end
        end
    end
```

引数を一つずつ吟味している。基本は nyagos.stat でディレクトリでない場合に「ディレクトリではない。」と出せばよいのだが、以下の例外があるので、それらに関してはスルーさせている。

- マイナスで始まるものはオプション
- 拡張子が .lnk（別のスクリプト cdlnk.lua ではショートカットの先に cd できるので）
- ルートディレクトリ (nyagos.stat がエラーになるので）
- 「..」「.」(示している先がルートディレクトリだったりすると nyagos.stat がエラーになる）

吟味した結果特にエラーがない場合は、元の cd を呼び出す。

```lua
    if success then
        if cd then
            cd(args)
        else
            args[0] = "__cd__"
            nyagos.exec(args)
        end
    end
end
-- vim:set fenc=utf8 --
```

変数「cd」は先に退避しておいた、オリジナルのエイリアスを格納したローカル変数。非nilなら、その関数をそのまま呼び出せばよし。nil なら、実際にディレクトリを変えればよろしい。が、`nyagos.exec('cd')`とかやると無限ループになる。かといって「`cd.exe`」などというものは存在しない。だが、**こんなこともあろうかと** `__cd__` などという内蔵コマンドの別名を…

