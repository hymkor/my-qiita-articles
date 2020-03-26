---
title: バッチファイルだけで wc -l の真似事
tags: batch
author: zetamatta
slide: false
---
```bat
@echo off
setlocal
:loop
if "%1" == "" exit /b
set COUNT=0
for /F %%I in (%1) do set /A COUNT+=1
echo %COUNT% %1
shift
goto loop
```

```
$ wc.cmd ~\_nya
45 C:\Opt\Share\_nya
```

CMD.EXE のバッチコマンドで変態なのは for だけじゃない！
set も十分変態だということですね、わかります。

