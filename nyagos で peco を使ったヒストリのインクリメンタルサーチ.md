---
title: nyagos で peco を使ったヒストリのインクリメンタルサーチ
tags: nyagos Peco
author: zetamatta
slide: false
---
一応、ネイティブで `C-r` のインクリメンタルサーチが実装されているけれども、peco を使った分かりやすいインクリメンタルサーチの手法を hisomura さんが提示されていたので、紹介せずにはいられなかった。

https://github.com/zetamatta/nyagos/issues/117 より：

```
nyagos.bindkey("C_R",
    function(this)
        local path = nyagos.getenv("APPDATA") .. '\\NYAOS_ORG\\nyagos.history'
        local result = nyagos.eval('type ' .. path .. ' | peco')
        this:call("CLEAR_SCREEN")
        return result
    end)
```

これ、いろんなもんで応用できるんじゃね :smile::smile: 

