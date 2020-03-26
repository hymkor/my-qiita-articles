---
title: Lua でイタレータを作ってみる(enumerate)
tags: Lua:5.1~5.2
author: zetamatta
slide: false
---
Python みたいなジェネレータはないけど、コルーチンがあるから簡単。

試しに、pair 等の結果に序列をつけるイタレータ(enumerate)を作ってみよう。

```lua
itertools = itertools or {}
local unpack_=(unpack or table.unpack)

function itertools.enumerate(func,stat,var0)
    return coroutine.wrap( function()
        local vars = { func(stat,var0) }
        local count = 1
        while vars[1] do
            coroutine.yield( count , unpack_(vars) )
            count = count + 1
            vars = { func(stat,unpack_(vars)) }
        end
    end)
end
```

```lua
for i,key,val in itertools.enumerate(pairs(foo)) do
    print(i,key,val)
end
```

初出：http://nyaos.org/d/index.cgi?p=%282012.12.12%29

