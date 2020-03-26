---
title: 【ポエム】NYAGOS 2017年ふりかえり
tags: Go nyagos
author: zetamatta
slide: false
---
[Nihongo Yet Another GOing Shell](https://github.com/zetamatta/nyagos) についての駄文です。関心ある方に読んでいただければと

* [参考：2017年冒頭に言ってたこと](https://qiita.com/zetamatta/items/3e83c7bfdfbe7fcc92b5)

----------

Luaデータ共有との戦い
=====================

＞ **完全に改善した！（← うさんくさい）** ＜

* 4.0
     * 全コマンド(=goroutine～スレッド)で 1-Lua インスタンスを共有
     * データ共有に問題はなかったが、クラッシュ多発
* 4.1
     * コマンドごとに Lua インスタンスを分離 ⇒ 動作が安定
     * 反面、グローバルとコマンド間でのデータ交換が不便に
         * テーブル`share[]`以外の変数・関数が、相互参照不能に
* 4.2 **[New!]**
     * Luaインスタンス作成時に、親のグローバル変数・関数を、子にフルコピー
         * 通称「**なんちゃってfork**」
     * ローカル変数や、クロージャーから参照される変数は複写できない
     * 子で設定したデータは、親や兄弟コマンドからは見えない
         * fork を持つ UNIX系/bin/sh でも、同じ現象が発生する（fork と同じ）
         * `share[]` の出番は減ったが、まだ必要

-------

foreach・ブロック-if
====================

＞ **念願の foreach ・ブロック-if を手に入れたぞ** ＜

![image.png](https://qiita-image-store.s3.amazonaws.com/0/29454/96f19b7e-b486-4a3b-e5e6-7bb2764a2ef4.png)

```
if /i "%PROCESSOR_ARCHITECTURE%" == "AMD64" then
    set "PATH+=%USERPROFILE%\Share\bin64"
end
```

* 今のところ nyaos-3000互換系(endifもendのエイリアスとして使える)
    * **CMD.EXE のように、丸括弧を使う構文に合わせたい**が、丸括弧の構文解析は難しい（まだ俺には）
* `_nyagos`(バッチ風) と `.nyagos`(Lua)、設定ファイルが複数あっても混乱するだけかも
    * また、迷いが出てきた ＞ Lua 一本でゆくか、バッチ構文併用でゆくか

--------

gawk と PowerShell
===============
はんぶん、ざつだん

* 「gawk にうまくコマンドを引き渡せない」という問題はその後発生していない
     * そもそも俺が使ってないからかな！
     * gawk が使える CSV って、要素に改行とかカンマ含められないから…（震え声）
* **PowerShell をよく使うようになった**
     * CSVだけでなく、Excelの読み書きも出来るしな！
     * 「`-ExecutionPolicy RemoteSigned`」が面倒。エイリアスできるが、スクリプトだけ手軽に他人に渡しにくい
     * Write-Output 、画面幅単位で勝手に改行コードを出すの、なんとかなりませんか！（`[System.Console]::WriteLine` でも使えと申すか？）
* **ビルドスクリプトを「バッチ埋め込み型PowerShell」にした**
     * 事実上、なんでもできる
     * `make get-lua64` とか `make get-lua32` ⇒ lua53.dll をダウンロードできるよ
     * `go-versioninfo.exe` がなければ、自動的にダウンロードッ

---------

バージョン・ブランチ・配布管理
============================

＞ **一人開発なので、手間暇をかけすぎても、報われない。バランス大切** ＜

4.2 より

* 安定版(master)は「バイナリ」配布する
* 開発版(develop)は、開発ブランチを別途設けるだけで、いちいちバイナリ配布しない
* 双方共通の不具合が出たら、まず安定版で修正し、それをdevelopへmergeする
     * **rebase すると、github緑化詐欺になりそうだった**から
 
* 一人開発の場合、**手間暇と品質のバランス**的に、この方式が一番かも
     * 過去、他のツールで開発版もバイナリを出していたが、結構たいへんだった
     * 挑戦的な改変をきちんとわけて、安定版もいつでも公開しやすくなった

----

新たな組み込みエンジンの模索
==========================

別のツール([`expect for Command Prompt`](https://github.com/zetamatta/expect))開発で、GopherLua を試用

* これまで使わなかった理由
    * パターンマッチ機能が正規表現になっている
    * 外部のLua用のライブラリが使えない
* 現在：使わない理由があまりない
    * Lua本家互換のパターンマッチになっている（ように見える）
    * そもそも、外部のLuaライブラリ、ほとんど使ってない
 
少しずつ lua53.dll 依存コードの集約を進めているが、量がまだ多いため、GopherLua 切り替えはまだそうとう先に

-------

ユーザの利用スタイルの多様性を全て追えない
=======================================

* 「画面幅が大きいと、画面が乱れる」
     * MS-API の画面サイズを保持する変数のサイズが `signed short` 
         * うっかり掛け算をそのまますると 32767 を超えてしまい、オーバーフロー。
         * Ctrl-L で画面を奇麗に消しきれない状況に
     * ユーザには画面幅が多いという自覚がない。開発者は普段80桁でやってるから、ターミナルの問題に見えてしまう
     * **NYAGOS 4.2.2_2 にて解消**

* 「start が動かない」
     * 「%PATH% をたどってくれない」という意味だが、分からなかった。
     * CMD.EXE の start は %PATH% を追いかけていたのかー（「`start ‘which COMMANDNAME‘`」とかやってたから気づかなかった）
     * **NYAGOS 4.2.3_1 で解消**

----------

その他
======
* go-bindata は非推奨のムードなので、使用をやめた
    * `nyagos.d/` 以下を内蔵していたが、大して起動時間短縮に結び付いていなかった

* UTF8 と ANSI (日本語環境ならSJIS)の混在は「 **UTF8 として妥当ならUTF8、アウトなら ANSI**」という安直な方法で案外OK
    * `type`がタイニー文字コード変換コマンドに
* その他の自作 UNIX 風コマンド(diskfree、diskused）を組み込み
    * 抱き合わせ？
    * 他（[seek](https://github.com/zetamatta/seek)、[expect](https://github.com/zetamatta/expect)、[tail](https://github.com/zetamatta/tail)、[cure](https://github.com/zetamatta/cure)、[sponge](https://github.com/zetamatta/sponge)、[findo](https://github.com/zetamatta/findo)）も混ぜたいが、自重

-----

2018年への抱負
=============

＞ らいねん、かんがえる！ ＜

つまり、続くと？

