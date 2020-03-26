---
title: EXEファイルのバージョン情報を表示(IronPython)
tags: IronPython:2.7.4 .NET:4.0
author: zetamatta
slide: false
---
すまん、PowerShell はサポート範囲外なんだ。それゆえ、バッチ＋IronPython で：

```python:showver.cmd
@ipy -x "%~f0" %1 %2 %3 %4 %5 %6 %7 %8 %9 & exit /b

import System
import sys

for path1 in sys.argv[1:]:
    vi = System.Diagnostics.FileVersionInfo.GetVersionInfo(path1)
    if vi is None:
        continue
    print vi.FileVersion,vi.ProductVersion

# vim:set ft=python:
```

実行結果：

```
<MARKEDONE:~/bin>
✏ showver lua52.dll
5.2.3 5.2.3
```

