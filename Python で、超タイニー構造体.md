---
title: Python で、超タイニー構造体
tags: Python:2.7
author: zetamatta
slide: false
---
* タプル → 何番目が何か、よく忘れる
* 連想配列 → メンバー名を引用符で囲むのがメンドイ
* namedtupple → １行とはいえ、型をいちいち作るのがメンドイ。一回限りしか使わんパターンもあるし

いちいち別途「型」を作らずに、メンバーを引用符なしの名前でアクセスできたらいいな！

```struct.py
class Struct(object):
    def __init__(self,**kargs):
        for key,val in kargs.iteritems():
            setattr(self,key,val)

if __name__ == '__main__':
    t = Struct(a=10,b=20,c=30)
    t.c = "<30>"
    print t.a, t.b, t.c
```

```
<MARKEDONE:~>
✏ ipy struct.py
10 20 <30>
<MARKEDONE:~>
✏
```

たぶん、もっといい方法があるんだろうねぇ。亡者プログラマーだから、いろいろ忘れたよ。

