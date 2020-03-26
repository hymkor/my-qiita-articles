---
title: JScript をバッチファイル内に埋め込んでみる(shebangもどき)
tags: JScript bat
author: zetamatta
slide: false
---
できたけど、みっともない…

```javascript:foo.cmd
rem = 0 /* 
@cscript //E:JScript //nologo "%~f0" %1 %2 %3 %4 %5 %6 %7 %8 %9 & exit /b
*/
WScript.Echo("ahaha")

// vim:set filetype=javascript:
```

出力

```
<HAYAMA-PC:~>
✏ foo.cmd

[36;40;1m<HAYAMA-PC:C:\Users\Hayama>
rem = 0 /*
ahaha
```

この要らぬ出力、なんとかならんかのー。rem を @rem とかけるだけでも何とかなるんじゃが…

