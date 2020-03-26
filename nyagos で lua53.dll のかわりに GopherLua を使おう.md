---
title: nyagos で lua53.dll のかわりに GopherLua を使おう
tags: Go Lua nyagos ポエム
author: zetamatta
slide: false
---
[Using GopherLua instead of lua53.dll · Issue #300 · zetamatta/nyagos](https://github.com/zetamatta/nyagos/issues/300)

lua53.dll にはいくつか問題があります。

- MinGW gcc 6 でビルドされた lua53.dll は `LIBGCC_S_DW2-1.DLL` を必要とします。
   - [luaBinaries](http://luabinaries.sourceforge.net/index.html) でダウンロードできる lua53.dll は MinGW gcc 4 や Visual C++ でビルドされているようです。
   - GCC のstatic-linkオプションは、EXEファイルのビルドにおいて働くもので、DLL のビルドには効きません。
- appveyor での自動ビルド時、luaBinaries から lua53.dll をダウンロードしていますが、度々失敗しています。

また、nyagos で lua53.dll を用いたサードパーティライブラリも滅多に必要とされないようです。

ならば、lua53.dll のかわりに [GopherLua](https://github.com/yuin/gopher-lua) を使うべきではないでしょうか。
計画は始まっています。

---

ということで [NYAGOS](https://github.com/zetamatta/nyagos) は今、Lua エンジンを lua53.dll から GopherLua へ切り替え中です。（完成したら NYAGOS 4.3 になるでしょう）

仕掛中のソースは develop ブランチの https://github.com/zetamatta/nyagos/tree/develop/gopherSh 以下にぶら下がっています。エンジン自体は既に GopherLua に変わっていますが、幾つかの機能（補完のフックと、OLE呼び出し）の移植が済んでいません。進捗的には 2018年5月の連休までにα版がリリースできたらいいなというところです。

ご期待ください
（また、別途、lua53.dll と GopherLua の使い勝手の違いもリポートします）

