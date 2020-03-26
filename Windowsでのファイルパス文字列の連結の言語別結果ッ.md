---
title: Windowsでのファイルパス文字列の連結の言語別結果ッ
tags: Windows Go VC++ .NET WSH
author: zetamatta
slide: false
---
ファイルパス文字列の連結は人それぞれで「あるべき仕様」の見解があると思いますが、基本的に自分は

- パスA + パスB の結果は、「cd A」した状態で B が指すパスを示すものと等価

であってほしいと考えます。具体的には

（例）
- `foo` + `bar` = `foo\bar` (相対パスどうし）
- `foo` + `\bar` = `\bar` (二項目が絶対パスだから）
- `c:foo` + `\bar` = `c:\bar` （二項目は絶対パスだが、ドライブレターがないので、そこだけ一項目が生きる）

で、各言語での結果を確認しました。



Windows Scripting Host(JScript)
===============================

```javascript
var fsObj = new ActiveXObject("Scripting.FileSystemObject");
WScript.Echo( fsObj.BuildPath("c:\\foo","\\bar") );
```

結果 → `c:\foo\bar` : Oh...

.NET Framework(PowerShell)
==========================

```powershell
Write-Host([IO.Path]::Combine("c:\foo","\bar"))
```

結果 → `\bar` : 惜しい。なぜ、そこでドライブレターを消す！まぁ、でも、許容できなくはない

Go
==

```go
package main

import "path/filepath"

func main(){
	print( filepath.Join(`c:\foo`,`\bar`) , "\n")
}
```

結果 → `c:\foo\bar` : orz

VC++
====

```cpp
#include "atlpath.h"
 /* 中略 */
CPath path1;
path1.Combine(_T("c:\\foo"), _T("\\bar"));

_fputts((LPCTSTR)path1, stdout);;
(void)getchar();
```

結果 → `c:\bar` : よりにもよって、お前が一番（俺にとって）正しい結果を返すか！


とりあえず、以上
（また、他の言語で試せる機会があったら追記します）

