---
title: [傾向と対策] なぜ、Windows 8 から多くのコンソールアプリケーションで日本語が入力できなくなったのか
tags: Windows8:8 nyaos:3
author: zetamatta
slide: false
---
- 本記事は [nyaos.org](http://nyaos.org/d/index.cgi?p=(2013.03.25)+1823) より転載したものです。

Windows 8 になってから、vim , NYAOS といった多くのコンソールアプリケーションで、 日本語が入力できなくなったようです。 

NYAOS は2012年12月頃に、Windows 8 のユーザさんの御協力を得て、Windows 8 対応を行ったのですが、見返してみると Bitbucket に [issue](https://bitbucket.org/zetamatta/nyaos3000/issue/16/windows-8-ime) は起案したものの、文書として Windows 8になって何が変わっていたかの説明をしていなかったようです。

まだ、世の中には Windows 8 に対応していないコンソールアプリケーションの方が多いので、ここで何がどうなっていたかを解説しておきたいと思います。

# 原因は API の使われ方が変わってしまっていたことにある。

コンソールでの1文字キーボード入力の API に [ReadConsoleInput](http://msdn.microsoft.com/ja-jp/library/cc429661.aspx) というものがあります。従来のアプリケーションは、このAPI1回のコールで、1バイトの文字コードを取得していました。

ところが、Windows 8 になってから、API1回のコールで、1文字分(つまり ShiftJIS だと2バイト)の文字コードが得られるようになってしまったようなのです。

そのため、Windows 7 までのコンソールアプリケーションを Windows 8 で動かすと、ことごとく ShiftJIS の2バイト目を落としてしまって、正常な日本語が入力できなくなったのです。

# API そのものは変わっていない

実のところ、API そのものの仕様は一切変わっていません。 MSDNの [ReadConsoleInput](http://msdn.microsoft.com/ja-jp/library/cc429661.aspx)を見ていただくと、 [ReadConsoleInput](http://msdn.microsoft.com/ja-jp/library/cc429661.aspx) は、キー入力専用APIというわけではなく、 コンソールで起きたイベント情報を得るAPIとなっており、 元々一度に複数レコードのイベント情報を取得できるようになっていました。

ところが、なぜか 7 まではShiftJIS 1文字の入力を受けとった時でも、 上位バイト下位バイトを2回の API コールに分けて受信するという形になっていたのです。 そのため、多くのコンソールアプリケーションがバイト数分 [ReadConsoleInput](http://msdn.microsoft.com/ja-jp/library/cc429661.aspx) を呼び出して、 最初の1レコードだけを取得するという形で実装されていました。

# 対応するにはどうすればいいか

[ReadConsoleInput](http://msdn.microsoft.com/ja-jp/library/cc429661.aspx) で受信するイベントのバッファを大目にとって、 全てのレコードをちゃんと受取側で処理するようにすれば Ok です。

参考になるかどうか分かりませんが、NYAOS だと、[getch_msvc.cpp](https://bitbucket.org/zetamatta/nyaos3000/src/de745b698e8b6f664a52afd9d7b2e8f37aa1f2ed/getch_msvc.cpp?at=default) というソースで、この複数レコード対応を行いました。

このソースファイル自体は元々[lukewarmさんからいただいたもの](http://d.hatena.ne.jp/lukewarm/20091204#p2)ですが、Public Domain とおっしゃっているので、多分、転用いただいても問題ないと思います。

まだ、Windows 8 は少数派だとは思いますが、 多くのコンソールアプリケーションで、きちんと日本語入力対応が進むことを願います。

