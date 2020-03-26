---
title: Git で過去の Commit に一時的に戻り、作業後、また最新のCommitを復活させる
tags: Git
author: zetamatta
slide: false
---
Mercurial だと　`hg update -r チェンジセットID` で済むんだが、Git ではどうすればいいんだろう。

ググったところ、`git checkout コミット名` でいいらしい。

ただ、そのままやると、`git log`で最近のコミットが見えなくなってしまうので、焦る焦る焦る焦る焦る焦る、お前ら焦らないか、俺は焦った。
(いや、一応 bitbucket にバックアップは取っているが、最新Commitについては自信がなかったのだ)

だが、実は一時的な無名ブランチが作られて、そっちへ切り替わっているだけのようだ。

```
$ git branch
* (detached from 5624b10)
  master
```

`git checkout master` したら、この一時ブランチも消えて、元に戻った。

```
$ git branch
* master
```

