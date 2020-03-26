---
title: 「git new」コマンドをでっち上げる（Windowsでgitのサブコマンドを作る）
tags: Git
author: zetamatta
slide: false
---
自分の場合、新規レポジトリを作る際、いつも定番的に実行しなければいけない作業がある

* `git init`
* 最初に空コミット作成
* 改行コード変換の無効設定
* 日本語ファイル名はそのまま表示させる
* メールアドレス等の設定

次の記事によると、「`git-XXXX`」という実行ファイルを作ると、`git XXXX` というサブコマンドを作成できるようだ。

* [便利な「git-サブコマンド」を作成する - Qiita](https://qiita.com/b4b4r07/items/6b76a5f969231e5e9748)

Windowsなので `git-new.cmd` というバッチファイルを作成して、自分の定番初期化作業を一括化してみよう。

```git-new.cmd
git init 
git commit -m "empty" --allow-empty
echo "* -text" > .gitattributes
git add .gitattributes 
git commit -m "Add .gitattributes as autocrlf=false"
git config --local core.quotepath false
git config --local user.email zetamatta@example.com 
git config --local user.name  zetamatta
```

テスト

```shell-session
$ git new
git: 'new' is not a git command. See 'git --help'.

Did you mean this?
        notes
exit status 1
```

**だめでした！**

EXEなどの実行ファイルでないとダメなのか。でも、スクリプトレベルのものをいちいち実行ファイル作るのは面倒すぎるなぁ。Linux だとシェルスクリプトが大丈夫なのに…**あ**

```git-new
#!/bin/sh

if [ ! -e .git ] ; then
    git init 
    git commit -m "empty" --allow-empty
fi
if [ ! -e .gitattributes ] ; then
    echo "* -text" > .gitattributes
    git add .gitattributes 
    git commit -m "Add .gitattributes as autocrlf=false"
fi
git config --local core.quotepath false
git config --local user.email zetamatta@example.com 
git config --local user.name  zetamatta
```

(最終版なので、いろいろ機能追加してますが、気にしないでください）

Git for Windows は MSYS ベースで動作しているので、普通に「`git-new`」という拡張子無しのテキストファイルを %PATH% 上において、シェルスクリプトを書けば Ok でした！

```shell-session
$ git new
Initialized empty Git repository in C:/Users/Hayama/Share/cmds/.git/
[master (root-commit) 6b0d022] empty
[master f9ca464] Add .gitattributes as autocrlf=false
 1 file changed, 1 insertion(+)
 create mode 100644 .gitattributes
```

以上

