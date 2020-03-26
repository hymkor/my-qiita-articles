---
title: go-ole で Excel を操作した場合と "tealeg/xlsx" で直接 xlsx ファイルを作った場合の違い
tags: Go Excel office
author: zetamatta
slide: false
---
以前、

* [CSV を「安全」に Excel に読み込む Go プログラムを書いたー（pipe2excel.exe）](https://qiita.com/zetamatta/items/d6e6b1e09bbbbba18d90)

という記事にある pipe2excel.exe というCSVのExcel 変換プログラムを作ったわけだが…

これ、巨大な CSV の時にあまりに遅いので、"[github.com/tealeg/xlsx](https://github.com/tealeg/xlsx)" というライブラリで直接 xlsx を出力できるようにしてみた（`-o`オプションを使う）

**＞ はやい ＜**

とっても速い。一瞬だ。考えてみたら go-ole だとセル１マス書くだけでも Excel と COM 通信をするわけだから、そりゃそうだ。ただ、セルのデフォルト状態がいろいろ違った。

* フォントが MS-Pゴシック でなかった
* 改行を含む場合に自動で折り返し状態にならなかった
* 縦方向がセンタリングされなかった。

これらはプログラム的に [go-ole](https://github.com/go-ole/go-ole) 版と互換動作するよう修正した。これだけ書くと、"[github.com/tealeg/xlsx](https://github.com/tealeg/xlsx)"には非のうちどころがないように思えるが、一応注意事項を記しておこう。

* "[github.com/tealeg/xlsx](https://github.com/tealeg/xlsx)" は「既存の xlsx を読む」「新規の xlsx を作成する」場合はよいが、「既存の xlsx に追記・更新する」場合、画像とか諸々消えてしまう（らしい）

この場合は [go-ole](https://github.com/go-ole/go-ole) を使うか、あるいは "[github.com/loadoff/excl](https://github.com/loadoff/excl)" を使う方がよさそうだ。"[github.com/loadoff/excl](https://github.com/loadoff/excl)" は検証していないが、作者の人自ら紹介記事（[Go言語でExcel操作ライブラリを書いてみた - Qiita](https://qiita.com/tebakane/items/2f2ed2558357c274c478)）を書いているので、そちらを参照いただきたい。

* **xls（Excel 2007 より過去の Excel で読み書きできるフォーマット）を作成できない**

まぁ、ええやろ、xls は。Excel の方に xlsx を読み書きするアドオンを入れてもらった方がよろしい

* **go-ole だと、腐っても Excel 本体を動かしているので、おかしなデータを作成してしまう可能性が小さい**

以上

