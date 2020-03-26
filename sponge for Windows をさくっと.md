---
title: sponge for Windows をさくっと
tags: Windows Go moreutils
author: zetamatta
slide: false
---
[moreutils](http://qiita.com/cuzic/items/1837cfe6572cbd3f09c2) の sponge はいいなぁ…ということで、Windows版をさくっと Go で書いてみました。

- https://github.com/zetamatta/sponge

sponge とは、標準入力を「全て」読み取り終えてから、その内容を引数のファイルに出力する moreutils の１コマンドで、一つのパイプラインの中でファイルの差し替えを行うことができるツールです。

具体的には「`cat -n foo.txt | sponge foo.txt`」などということができます。「cat -n foo.txt > foo.txt」だと、foo.txt が壊れてしまうところです。

実装上の迷いどころは以下のとおりでした。本家のソースは読んでないんですが、そちらの仕様はどうなんでしょうね？（← 読めよ）

- 複数のファイル名を指定した場合
     - 挙げられている全てのファイルに標準入力の内容を出す
- 標準入力のスプール先
     - 出力ファイル名＋「.sponge」という一時ファイルを作って、標準入力がクローズ次第、差し替えるという方式にした。（同一ディレクトリにしたのは、同一ディスクの方がファイル差し替えコストが低いことを見込んで）
     - メモリにスプールする方式もあったが、メモリ不足になる可能性もあるのと、どうせ最後に出力するのであれば、あまりパフォーマンス上のメリットもない

