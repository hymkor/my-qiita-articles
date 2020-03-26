---
title: /bin/sh に慣れた人に贈る CMD バッチの書き方
tags: バッチファイル バッチ cmd.exe cmd Windows
author: zetamatta
slide: false
---
ここに書いてあるようなことはだいたい「CMD.EXE /?」「for /?」で出るんだけど、長いので sh 経験者の観点でかいつまんでみた。

コマンド出力の引用
================

```bash
ls | while read line ; do echo ">>$line" ; done
```
には

```bat
for /F %%I IN ('dir /b') DO echo ">> %%I"
```

が対応する（丸括弧の内側にシングルクォートがあることに注意）

逆クォート相当のものも１行程度なら

```bat
for /F %%I IN ('echo AHAHA') DO SET "OUTPUT=%%I"
```

と出来なくもない。

for は頑張れば AWK に近いことも出来るので、「for /?」を見ることをお勧めする。

basename と dirname
===================

`dirname` 相当は次のような感じで出来る。

```bat
[c:tmp]
$ type dirname.cmd
@echo off
for %%I IN ( c:\path\to\file ) do set "DIRNAME=%%~dpI"
echo %DIRNAME%
[c:tmp]
$ dirname
c:\path\to\
```

`basename` の場合は %%~dp のところを %%~nx か %%~n にすれば OK。

%~ の後の「英小文字」は変数の中身を加工する意味があるので、変数名自身は「英大文字」にしておいた方がよい。


find -exec
==========

これ、自分の %HOME%\bin に置いている、%HOME%以下のツリーにあるゴミファイルを片づけるバッチ。

```bat
cd %~dp0..
for /R %%I in (*.orig *~ svn-commit*.tmp .*.swp *.rej ) do (del /P "%%I")
```

最初の「%~dp0..」はバッチが置いてあるディレクトリの親ディレクトリの意味。

forfiles.exe (追記:2015.08.10) 
-----------------

最近の Windows では forfiles.exe という find 相当のコマンドも標準装備されている

```
forfiles /S /M *.cpp /C "cmd /c echo @path"
```

こちらは一度に /M オプションで指定できるパターンは一つだけのようだ。
だが、ファイル名の展開用マクロ(@path,@isdir)がいろいろ使える

ブロックIF
=========

古い単語出た。IF や FOR は１行で書いてしまわないといけないかというと、丸括弧を使うと複数行に展開できる。

```bat
if "%1" == "a" (
    echo arg is a
    echo aaaaaaaa
) else (
    echo arg is else
    echo bbbbbbbb
)
```

ただし、環境変数の展開のタイミングが if 文実行前なので、そこは工夫が必要だ。
（詳しくは `SET /?` を参照）

(追記:2018.09.15)
 `SETLOCAL ENABLEDELAYEDEXPANSION` を使えば「`!環境変数名!`」形式の遅延環境変数の展開が使える（詳しくは `SETLOCAL /?` のヘルプを見よう）。ただし、自分は綴りを覚えられないので、FOR の中から`CALL :ラベル "%%I"` を使って誤魔化すことが多い。

シェルレベルでの二重起動防止
=========================

* ps は tasklist で代用可能
* grep は findstr で代用可能
* findstr の成否は ERRORLEVEL に出る
* IF ERRORLEVEL 1 くらいは知ってるよね

あとは分かるな？

```bat
tasklist | findstr ttermpro.exe
if errorlevel 1 (
   rem 起動していない
)else(
   rem 起動してる
)
```

where.exe (追記:2018.09.15)
=========================

C:\Windows\System32\where.exe は which と find の両方の機能がある。

オプションなしだと、全PATHを検索する which のような機能になる。

```
$ where nyagos.exe
C:\Users\hymko\Share\bin64\nyagos.exe
C:\Users\hymko\Share\bin\nyagos.exe
C:\Users\hymko\go\bin\nyagos.exe
```

/R オプションをつけると find に近くなる。

```
$ where /R . *.go *.md
C:\Users\hymko\go\src\github.com\zetamatta\findo\findo.go
C:\Users\hymko\go\src\github.com\zetamatta\findo\hoge and hoge.md
C:\Users\hymko\go\src\github.com\zetamatta\findo\readme.md
C:\Users\hymko\go\src\github.com\zetamatta\findo\system_linux.go
C:\Users\hymko\go\src\github.com\zetamatta\findo\system_windows.go
```

タイムスタンプの取得 (追記:2018.09.15)
====================================

分以上（まだ、お手軽・早い）
-------------------------

```
$ type f.cmd
@echo off
for %%I in (readme.md) do set "STAMP=%%~tI"
echo %STAMP%
$ f
2018/09/13 22:35
```

秒まで(forfiles版)
-----------------

```
$ type m.cmd
@echo off
for /F %%I in ('forfiles /P "%~dp1." /M "%~nx1" /C "cmd.exe /c echo @fdate-@ftime"') do set "STAMP=%%I"
echo %STAMP%
$ m readme.md
2018/09/13-22:35:40
```

※ 0時～9時までは、時部分が一桁になるので注意が必要

秒まで(where版)
--------------

```
$ type x.cmd
@echo off
for /F "tokens=2,3" %%I in ('where /R . /T "%1"') do set STAMP=%%I-%%J
echo %STAMP%
$ x readme.md
2018/09/13-22:35:40
```

※ 0時～9時までは、時部分が一桁になるので注意が必要

0～9時のタイムスタンプに対する対策(修正:2018.09.24)
------------------------------------------------

where でも forfiles でも時間部分が一桁の場合、タイムスタンプの比較が不具合を起こす。やむをえないので正規表現で判別するというベタな方法で対応する。

```
for /F "tokens=2,3" %%I in ('where /R . /T "%~1"') do set STAMP=%%I_%%J
echo %STAMP% | findstr _[0-9]: > nul && set STAMP=%STAMP:_=_0%
```

文字列置換 (追記:2018.09.24)
===========================
bash だと

```
$ FOO=abcdef
$ echo ${FOO/cd/12}
ab12ef
```

という置換が使える。CMD.exe でも似たようなことが出来る。

```
$ set FOO=abcdef
$ echo %FOO:cd=12%
ab12ef
```

改行しない出力 (追記:2020.01.19)
=============

[DOSコマンドで、好きな形式に「パスのコピー」する - Qiita](https://qiita.com/zarukishi/items/6181c1a0390444d00002)より

`echo -n MESSAGE` 相当の改行しない echo は

```
SET /P ="MESSAGE" < NUL
```

で実現できるようだ。この SET は本来 `SET /P VAR=PROMPT` という形式で、PROMPT を表示した後、ユーザが入力した値を環境変数 VAR に設定するコマンドだが、環境変数名は省略できる。さらに標準入力を NUL でリダイレクトすることで、ユーザ入力自体も素通りさせられる。

