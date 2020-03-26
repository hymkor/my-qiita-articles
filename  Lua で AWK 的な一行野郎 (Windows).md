---
title:  Lua で AWK 的な一行野郎 (Windows)
tags: Lua:5.2
author: zetamatta
slide: false
---
Windowsで、AWKを使おうとすると、コードを囲むのも、コード中の文字列も両方ダブルクォーテーションで、いろいろしんどいので、大好きな Lua で AWK 的な一行野郎を書けるようにしてみた。
（いや、シングルクォーテーションでコード囲めるように手を加えてある AWK も多いんだが、NYAOSがうにゃうにゃ)

```lua:luawk.cmd
::rem:: --[[ vim:set ft=lua: 
@lua "%~f0" %1 %2 %3 %4 %5 %6 %7 %8 %9 & exit /b 0
]]--

-- requires Lua 5.2

if #arg < 2 then
    print(string.format('Usage: %s "ONELINEAR" {FILENAMES}', arg[0]))
    os.exit(0)
end

local function split(line)
    local S = {}
    for p in string.gmatch(line,"%S+") do
        S[#S+1] = p
    end
    S[0] = line
    return S
end

local onelinear = assert( load(arg[1] ) )

NR = 0
if #arg >= 2 then
    for i=2,#arg do
        FNR = 0
        FILENAME = arg[i]
        for line in io.lines(FILENAME) do
            NR = NR + 1
            FNR = FNR + 1
            S = split(line)
            onelinear()
        end
    end
else
    FNR = 0
    FILENAME=""
    for line in io.lines() do
        NR = NR + 1
        FNR = FNR + 1
        S = split(line)
        onelinear()
    end
end
```

だいたい、こんな感じで使う。

```
luawk "if #S>=1 then print('\tPageData.Position.'..S[1]..',') end" d2kcustomize63320.txt 
```

``$0`` , ``$1`` , ``$2`` … は ``S[0]`` , ``S[1]`` , ``S[2]``…。
NR,FNR,FILENAME といった組込み変数はサポートした。

が、正規表現はやっぱり AWK ほど手軽にはかけないねぇ。
