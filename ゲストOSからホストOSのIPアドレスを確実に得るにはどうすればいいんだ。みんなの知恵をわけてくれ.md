---
title: ゲストOSからホストOSのIPアドレスを確実に得るにはどうすればいいんだ。みんなの知恵をわけてくれ
tags: virutalbox Windows
author: zetamatta
slide: false
---
ホストOS の ssh 転送をゲストOSからも使いたいんですわ。

で、どうもネットワーク設定が「NAT」の場合はデフォルトゲートウェイがたいていホストOSのIPになっているみたいなので、とりあえず、こんな Lua バッチを書いてみた。一応、期待動作はするんですけどね (Windows7ゲスト on VirtualBox)

```lua:hostip.cmd
::rem:: --[[ vim:set ft=lua:
@lua "%~f0" %1 %2 %3 %4 %5 %6 %7 %8 %9 & exit /b
]]--

function getGateway()
    local ipconfig = io.popen("ipconfig","r")
    for line in ipconfig:lines() do
        local m = line:match("デフォルト.*%s([0-9]+%.[0-9]+%.[0-9]+%.[0-9]+)")
        if m then
            ipconfig:close()
            return m
        end
    end
    ipconfig:close()
end
print(getGateway())
```

```text:実行結果
✏ hostip
10.0.2.2
```

ホストOSのIPアドレスがデフォルトゲートウェイに設定されて、それを検索するのはいいんだけど、これじゃあ、どこでも動くというわけにはいかないよね？

* 検索キーワードが「デフォルト」って日本語はどうなのよ？
* 複数ネットワークアダプタがある時、これでいいの？
* これ、VirtualBox でしか試してないけど、VMware でも動けばなぁ（理由あって、両方併用してる。まだ試せてない）
* できたら、ゲストOSとホストOSを自力で区別したいなぁ（ホストだと 127.0.0.1 を返すようにしたい)

みんなの知恵を分けてくれ！

