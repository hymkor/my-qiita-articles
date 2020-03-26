---
title: Windows における su(sudo) 事情考察
tags: Windows
author: zetamatta
slide: false
---
su (sudo）とは
==============

suというと、通常は UNIX(Linux)で、ログアウトせずにユーザを切り替えるコマンドを指し、主に root (管理ユーザ）に切り替える用途に使われることが多い。そして、sudo は１コマンドだけユーザを切り替えて実行するコマンドだ。

最近、Windows でも su(sudo)的なコマンドが現れたが、これは常に管理ユーザで使われがちの Windows においては、ユーザを切り替えるというよりも、管理者権限を獲得するためのコマンドという位置づけとなっていることが多い。

本文書では、管理者権限を得てコマンドを実行する操作について、手法を紹介してみたい。

管理者権限があるかどうかのチェック
===============================

C++
----

advapi32.dll の `OpenProcessToken` と `GetTokenInformation` という API を組み合わせてチェックするという方法がある。詳しく解説しているページはこちら

* [或るプログラマの一生 » UAC 環境下で管理者権限で動いているかどうかのチェック (Windows/Win32API)](http://umezawa.dyndns.info/wordpress/?p=5191)

自分でも Go 言語で実装してみた。(nyagos'Lua からは`nyagos.elevated()`で利用できる）

* https://github.com/zetamatta/nyagos/blob/master/dos/elevation.go

バッチファイル
-------------

「`net session >nul 2>&1`」は管理者権限でないと ERRORLEVEL=2 で失敗するということを利用して判別できる。このテクニックは「[Office 365のデスクトップアプリケーションをバッチファイルで修復](http://qiita.com/stillalive/items/3889452338f4922fb1a8)」で用いられていたが、なかなか手軽でよいと思われる。


管理者権限を得るには
==================
C++
----

Windows の API で管理者権限を得るのは ShellExecute になる。

Windows ではプロセスを起動する有名どころの API としては CreateProcess と ShellExecute の２種類がある。CreateProcess は「普通」にプロセスを起動する API で、**パラメータがやたら多くて難しい**（← いいすぎ）他は特筆すべきことはない。たいていの言語の子プロセス起動ライブラリはこれを使用している。一方、ShellExecute はデスクトップからアイコンをクリックしたのと同じような形でコマンドを実行するような API である。

ShellExecute のパラメータは CreateProcess よりむしろ簡単で、次のような形となる。

https://msdn.microsoft.com/ja-jp/library/cc422072.aspx より引用：

```
HINSTANCE ShellExecute(
    HWND hwnd,              // 親ウィンドウのハンドル
    LPCTSTR lpVerb,         // 操作
    LPCTSTR lpFile,         // 操作対象のファイル
    LPCTSTR lpParameters,   // 操作のパラメータ
    LPCTSTR lpDirectory,    // 既定のディレクトリ
    INT nShowCmd            // 表示状態
);
```

lpVerb に入れる文字列で `L"open"` を与えると、普通の起動になるが、それを `L"runas"` と変えると、管理者として実行となる。これを利用した su/sudo はかなり多く、NYAGOS に内蔵している su / sudo もこれである

.NET Framework
--------------

```
Dim pinfo1 As New System.Diagnostics.ProcessStartInfo()
pinfo1.Verb = "RunAs" '*** This is for UAC mode ***
' 中略
Dim process As System.Diagnostics.Process =
       System.Diagnostics.Process.Start(pinfo1)
```

これを利用した su として以前 [wouldyou.exe](https://github.com/zetamatta/wouldyou)というものを作ったが、後述の powershell を使った方法の方で用が足りるので、今はほとんど使っていない。

バッチファイル＆PowerShell
------------------------

* **2018.01.19追記**
    * 下記より、[管理者権限でbatを実行したい時にやッた事](https://qiita.com/resolver/items/7187bd6ee8016ee5c741)で紹介された、「`powershell start-process 実行対象 -verb runas`」の方がより短いようです。

.NET Framework と同じ方法がとれるが、COM を使った方法の方がお手軽。COM を使っているので、当然ながら JScript/VBScript でも同じことができる。が、起動ロゴが出たりするので、やはり、PowerShell が最適解だろう。

```su.cmd
@setlocal
@if not "%1" == "" @set "ARG=/c %*"
powershell "(New-Object -Com Shell.Application).ShellExecute('cmd',$Env:ARG,'','runas')"
@endlocal
```

これのすごくよいところは１行コピペしてしまえば、それだけで用が足りる点だ
（バッチファイル１枚の中で完結する）。

真の sudo を目指して
===================

これらを利用して、su/sudo 的なものは比較的簡単に実現できる。自分はこれで満足していたが、コマンドプロンプトで実行する際、管理者権限を得たコマンドラインは別のコンソール窓になってしまうという点があり、これを使いにくく感じる向きも多いようだ。

ということで、それを解決した sudo も登場している。

* [wantora / sudo — Bitbucket](https://bitbucket.org/wantora/sudo)
* [mattn/sudo: sudo for windows](https://github.com/mattn/sudo) （解説記事：[Big Sky :: sudo コマンド書いた。](https://mattn.kaoriya.net/software/lang/go/20170614142801.html)）
* [msmania/sudo: SUDO for Windows](https://github.com/msmania/sudo)（解説記事：[SUDO for Windows | すなのかたまり](https://msmania.wordpress.com/2016/06/30/sudo-for-windows/)）
* [PowerShellでsudo - Qiita](http://qiita.com/twinkfrag/items/3afb9032fd73eabe09be)

実装の仕方を見ると、管理者権限で動くプロセスを別に立ち上げて、そちらとパイプラインを結んで、それまで使っていたコンソールに管理者権限で実行しているプロセスの標準入出力を引用しているようだ（← あまりソースを真剣に読んでいない）

なお、これらを NYAGOS で動かしたい時は NYAGOS内蔵の `sudo` を無効にするため、

* `~\.nyagos` に`nyagos.alias.sudo="sudo.exe"`

もしくは

* `~\_nyagos` に `alias sudo=sudo.exe`

としよう(`.exe`をつければ内蔵コマンドとみなされない)。nyagos の sudo を使わないのは、それが新しいコンソールを起動してしまうためだ（一応、廃止も検討している）

以上











