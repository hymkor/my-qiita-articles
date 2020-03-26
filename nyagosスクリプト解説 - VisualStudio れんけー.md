---
title: nyagosスクリプト解説 - VisualStudio れんけー
tags: nyagos
author: zetamatta
slide: false
---
設定
=====

自分の .nyagos はこんな感じになっている。

```Lua:.nyagos
local ProgramFiles="C:\\Program Files (x86)"
if not nyagos.stat(ProgramFiles) then
    ProgramFiles = "C:\\Program Files"
end

local function source(args)
    args[0] = "source"
    local fd = io.open(args[1])
    if not fd then
        return nil,"File Not Found"
    end
    fd:close()
    return nyagos.exec(args)
end

source{ nyagos.pathjoin(ProgramFiles,
        "Microsoft Visual Studio 10.0\\VC\\vcvarsall.bat"),
        "x86" }
```

ややこしいことをしているけど、要は

```
source "C:\Program Files\Microsoft Visual Studio 10.0\\VC\\vcvarsall.bat" "x86"
```

を実行すれば、環境変数が Visual Studio Command Prompt 相当になるっていうだけ。

活用
====

コマンドラインから Visual Studio を起動する
-----------------------------------------

```
open *.sln
```

要はソリューションファイルを開けばいいだけ

Visual Studio を起動せず、ビルドする
----------------------------------

こんな Makefile をつくったりー（ここでは TAB がスペース化されているので、コピペする時は注意）

```
build:
	devenv XXXXXXX.sln /build Release

debug_:
	devenv XXXXXXX.sln /build Debug

clean:
	devenv XXXXXXX.sln /clean Release
	devenv XXXXXXX.sln /clean Debug
```

マクロを使っているので、mingw32-make.exe でも nmake.exe でも、どっちでもいけるはず

