---
title: lua 5.1 for Windows で shebang もどき
tags: Lua
author: zetamatta
slide: false
---
- [インタプリタ無改造！ Lua on Windows で shebang](https://qiita.com/hymkor/items/4f0c1d4c3f1fa76d8e4c)

上記の方法は、ラベルがある Lua 5.2 以降でしか使えなかった。

GopherLua など、最も普及した「lua 5.1」で shebang（バッチファイルにスクリプトを埋め込んで、インタプリタ名の入力を省く）を書くには次のように、ワンライナー中にスクリプトの３行目以降をロードするコードを書く方法がある。

```
@lua5.1.exe -e "_,_,b=io.input([[%~f0]]):read('*l','*l','*a');assert(loadstring('\n'..b,[[%~f0]]))()"
@exit /b %ERRORLEVEL%

print('ahaha')
```

一応、エラー時の行番号もずれない（評価する前に頭に `\n\n` を追加して、読み飛ばた文の行位置の調整をしている）

