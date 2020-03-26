---
title: ここ（Windows）に「☭」という名前のファイルがあるじゃろ？
tags: Go:1.3
author: zetamatta
slide: false
---
中身も「☭」(UTF8)のはずなんじゃが、そのまま type すると

```
<C:\Opt\gosrc\nyagos>
$ type ☭
笘ｭ
```

だめ。そりゃ、chcp 932 のままだからの。では、nkf32 を使ってはどうか。

```
<C:\Opt\gosrc\nyagos>
$ nkf32 < ☭
笘ｭ
```

うん、ShiftJIS で表現できない文字だからのー。

だが、先日 go で作った [ANSI エスケープシーケンスをカットするだけの cat もどき](http://qiita.com/zetamatta/items/a3f421297970468c3dd9)を通すと、あら不思議

```
<C:\Opt\gosrc\nyagos>
$ ansistrip.exe < ☭
☭
```

ちゃんと表示されるんじゃ。goの標準ライブラリがUTF8をUTF16に変換してAPIに渡してくれるんじゃ。これだけでも go を使う価値ありじゃろ？

