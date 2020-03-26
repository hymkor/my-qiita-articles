---
title: strict.pm が無い環境でも一応エラーにならない use strict の書き方
tags: Perl:5.0
author: zetamatta
slide: false
---
具体的に言うと、@nifty の「[@homepage](http://homepage.nifty.com/service/)」（[LaCoocan](http://homepage.nifty.com/)じゃねーぞ）の Perl は **strict.pm すら無くて**だなぁ、今まで @homepage 向けにいちいち

```pl
# use strict; use warnings
```

などとコメントアウトしていた。だが、最初から

```pl
BEGIN{
    eval{ require 'strict.pm';   }; strict  ->import() unless $@;
    eval{ require 'warnings.pm'; }; warnings->import() unless $@;
}
```

等と書いておくと

* strict.pm がある環境では、use strict; 相当の動作をさせる
* strict.pm がない環境では、スルー

させることが可能だと気付いた。

とは言え、きょうび、そんな環境ほとんどないし、唯一と言っても過言ではない @homepage も2010年1月31日で新規受付終了してるし、これも完全にロストテクノロジーだなぁ。

