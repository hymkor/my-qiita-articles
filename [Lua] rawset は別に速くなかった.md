---
title: [Lua] rawset は別に速くなかった
tags: Lua:5.2
author: zetamatta
slide: false
---
# table.insert

```Lua:bench1.cmd
::rem:: --[[ vim:set ft=lua:
@lua "%~f0" %* & exit /b
]]--
t = {}
for i=1,100000 do
    table.insert(t,"x")
end
print(os.clock())
```

```
$ bench1
0.018
```

# 普通にテーブルアクセス

```Lua:bench2.cmd
::rem:: --[[ vim:set ft=lua:
@lua "%~f0" %* & exit /b
]]--
t = {}
for i=1,100000 do
    t[i] = "x"
end
print(os.clock())
```

```
$ bench2
0.006
```

# rawset

```Lua:bench3.cmd
::rem:: --[[ vim:set ft=lua:
@lua "%~f0" %* & exit /b
]]--
t = {}
for i=1,100000 do
    rawset(t,i,"x")
end
print(os.clock())
```

```
$ bench3
0.015
```

# 結論

- 関数コールが入るとやはり遅い
- 人間素直に生きよう
- そんなに気にするんだったら、LuaJIT 使えば？

