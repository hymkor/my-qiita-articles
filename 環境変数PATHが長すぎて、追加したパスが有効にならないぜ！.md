---
title: 環境変数PATHが長すぎて、追加したパスが有効にならないぜ！
tags: nyagos
author: zetamatta
slide: false
---
ならば削ればよかろうなのだッ。

グローバルな %PATH% から削ると問題でそうなので、コマンドラインから呼び出しそうにないパスだけ、.nyagos で削ればよいのだ。

```lua:%USERPRFILE%\.nyagos
-- Remove %PATH%
do
    local paths = {}
    for dir in string.gmatch(nyagos.env.path,"[^;]+") do
        if not string.find(dir,"Intel") and 
            not string.find(dir,"Lenovo",1,true) and
            not string.find(dir,"Skype",1,true) and
            not string.find(dir,"SQL Server",1,true) and
            not string.find(dir,"TypeScript",1,true) and
            not string.find(dir,"Performance",1,true) and
            not string.find(dir,"Windows Kits",1,true) then
            paths[#paths+1] = dir
        end
    end
    nyagos.env.path = table.concat(paths,";")
end
```

- 4.0.9 の nyagos では環境変数を nyagos.env.xxxxx といった形で「読書き」できる（大文字・小文字も区別しない）
- nyagos限定の記事でも、タグを「nyagos」だけに限れば、関心ない人のタイムライン汚さないよね（まぁ、ストック数はゼロだろうけど、気にしないようにしよう。それよりも「情報がない」とか言われる方がアレだから）

