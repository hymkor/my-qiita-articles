---
title: VB.Net の FileOpen 関数
tags: VB.Net:2010
author: zetamatta
slide: false
---
VB6 からVB.Netへコンバートしたソースなどでは FileOpen 関数がそのまま使われていることが多い。だが

**FileOpen関数のファイルハンドルは、アセンブリローカルだぞ**

それゆえ、ハンドル(Int32)を DLL に渡してもちゃんと機能しない。時間がちゃんとあれば、StreamWriter/Readerに書き換えておきたいものだな…

