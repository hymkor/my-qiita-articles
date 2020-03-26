---
title: NYAOS で「cd」した時のディレクトリを %HOME% や %USERPROFILE% 以外に
tags: nyaos:3
author: zetamatta
slide: false
---
自分は C:\Opt\SkyDrive を %HOME% にして、複数のシステムで共有しているんだけれども、ローカルのものは C:\Opt 以下に置くので、そっちに移動した方がいいんだよね

そういう場合は、_nya にこう書けばよろしい

```
cd{
    if "%1" == "" then
        __cd__ ~\..
    else
        __cd__ "%1"
    endif
}
```

```__cd__``` は、cd のシステム組み込みの別名。```cd{ … }``` はあまり知られていないが、関数機能という。

