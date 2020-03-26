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


![image](https://qiita-image-store.s3.amazonaws.com/0/29454/62d530c5-98b7-472f-2cd7-3f0e5a6b2ed8.png)

＞ **今、先方のサイトに issue 起案したから、ちょっと待ってて！** ＜

https://github.com/Maximus5/ConEmu/issues/1103

