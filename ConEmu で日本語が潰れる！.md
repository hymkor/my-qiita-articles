---
title: ConEmu で日本語が潰れる！
tags: Windows nyagos cmd conemu
author: zetamatta
slide: false
---
どうやら、画面幅を超えるテキストをCRLF無しで連続して出力すると、折り返し処理が走るタイミングでフォント幅の見積もり計算がおかしくなるようです。

```foo.cmd
echo echoがずれる問題日本語
echo echoがずれる問題日本語fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```


![image](a45fb037-925c-450b-a47f-e6a5d91d9edd.png)

＞ **今、先方のサイトに issue 起案したから、ちょっと待ってて！** ＜

https://github.com/Maximus5/ConEmu/issues/1103

