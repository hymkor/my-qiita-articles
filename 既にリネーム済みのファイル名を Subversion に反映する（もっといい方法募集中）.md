---
title: 既にリネーム済みのファイル名を Subversion に反映する（もっといい方法募集中）
tags: svn batch
author: zetamatta
slide: false
---
Mercurial だと ``hg rename -A`` というオプションがあるが、``svn`` にはなさそうなので、仕方なく、以下のようなバッチを作った。

```bat
rem @echo off

if not exist %2 goto usage
if     exist %1 goto usage

move %2 %1
svn move %1 %2
exit /b

:usage
    echo svn renamer like hg rename -A
```

もっといい方法はないものか…

