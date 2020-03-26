---
title: let のない Lisp処理系で lambda を let のかわりにつかう
tags: lisp AutoLISP AutoCAD BricsCAD
author: zetamatta
slide: false
---
無いものを嘆いていても仕方がないので、できることを模索しよう。

autolisp系のLispでは let が使えず、ローカル変数は、関数の頭でしか宣言できない。また、use strict 相当のものがないため、関数が長くなるとローカル変数の宣言漏れがないかどうかをチェックするのも（宣言するところと使用する場所が離れているため）結構たいへんだ。

そんな折、[let が lambda の糖衣構文である処理系もあるという話](http://www.shido.info/lisp/scheme6.html)を聞いた。なるほど、let は lambda の変形なのか。ということで、lambda で let 相当のことをやってみよう。

```test-lambda.lsp
(defun C:test-lambda (/ tmp)
  (setq tmp "<global value>")
  (princ "\nBefore=")
  (prin1 tmp)
  ((lambda (/ tmp) ; let local
     (setq tmp "<local value>")
     (princ "\nIn lambda=")
     (prin1 tmp)
  )) ; end local
  (princ "\nAfter=")
  (prin1 tmp)
  (princ)
)
```

思ったほど、見づらくない。lambda と同じ行に「`; let local`」と書くようにすれば、他の読み手にも分かってもらえそう。動作結果はどうだろう。

```
: TEST-LAMBDA

Before="<global value>"
In lambda="<local value>"
After="<global value>"
```

よしよし

