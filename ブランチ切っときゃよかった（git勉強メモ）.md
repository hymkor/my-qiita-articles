---
title: ブランチ切っときゃよかった（git勉強メモ）
tags: Git
author: zetamatta
slide: false
---
# 参考にしたページ

* [Gitでブランチを作るのを忘れてmasterにコミットしてしまったときの対処法](http://qiita.com/atskimura/items/a90dfa8bfc72e3657ef9)
* [今さら聞けないgit pushコマンド](http://shoma2da.hatenablog.com/entry/2014/03/08/234523)

# 経緯

[某ソフトウェア](http://github.com/zetamatta/nyagos)に試しに Lua を組み込んじゃったけど、「master」ブランチでやっちゃった。「mruby」にするかもしれんし、別ブランチにしときゃよかったなー

# やってみた

Lua を組み込む前の commit で、ブランチを作成する

```text
git branch Lua 9ca44d6dce92e2c27476704feda38789fb280c1a
```

(HEAD表記はズレてたら、こわいんで。しかし、Lua を組み込む前なのに、なぜ、ここで Lua というブランチ名にするのだ＞俺)

master と Lua を入れ替える

```text
$ git branch -m master lua_
$ git branch
* Lua
  lua_
$ git branch -m Lua master
$ git branch -m lua_ Lua
$ git branch
  Lua
* master
```

git log を見る限り、これでよい気がした。これから、github に upしようと思ったが…

# まずそうだ。

```text
$ git checkout Lua
Switched to branch 'Lua'
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
[C:nyagos]
$ git checkout master
Switched to branch 'master'
```

なんか、このまま git push --all すると、まずそうだ。なんとかしないと。

```text
git push -u origin Lua:Lua
```

こっちはうまいこといった。

```text
$ git push -u origin master:master
Username for 'https://github.com': zetamatta
Password for 'https://zetamatta@github.com':
To https://github.com/zetamatta/nyagos.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/zetamatta/nyagos.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

こっちは既に Up 済みの commit があったので、失敗した。

```text$ git push -u -f origin master:master
Username for 'https://github.com': zetamatta
Password for 'https://zetamatta@github.com':
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/zetamatta/nyagos.git
 + ef5ff32...9ca44d6 master -> master (forced update)
Branch master set up to track remote branch master from origin.
```

-f オプションで強制したら、いけた。そんな修正で大丈夫か？

# お願い

足りんとことか、まずいとこあったら、教えていただけると幸いです。

