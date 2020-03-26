---
title: Visual Studio 201x の devenv.com はどこにある？
tags: VisualStudio
author: zetamatta
slide: false
---
devenv.com は Visual Studio の IDE を起動する実行ファイルだが、オプションでコマンドラインからのビルドを行うことができる。

Microsoft はコマンドラインビルドには、devenv.com ではなく MSBuild を使うべきと言っている。だが、VB.Net の DLL をビルドする時、なぜか拡張子が .dll.dll という名前で生成されてしまう現象が過去確認されている（Visual Studio 2013か2015くらいの頃だったと思う。今は直っているかもしれないが）。リネームすればよいが、それもあまりスマートとは思えないため、自分は devenv.com を使う方が無難と判断している。

devenv.com のフルパスについては過去グローバルな環境変数で特定できていたが、2017以降は廃止されている。通常は Command Prompt for vsXX などのショートカットからコマンドプロンプトを起動すれば devenv.com へパスが通る。だが、これで呼び出されるバッチファイルは設定される環境変数がてんこ盛りで起動が遅い。一方で devenv.com 自身は PATH さえ通っていれば利用に支障はないので、このショートカットをあまり利用したくない。

ということで、バージョンごとの devenv.com の場所の特定方法をまとめた。
（以下は 「[Visual Studio のコマンドラインクライアントのラッパー vf1s を作った](http://zetamatta.hatenablog.com/entry/2019/06/22/160031)」よりの抜粋となる）

Visual Studio 2010～2015
========================

環境変数に Tools フォルダーにパスが格納されている。

* 2015 → `%VS140COMNTOOLS%`
* 2013 → `%VS120COMNTOOLS%`
* 2010 → `%VS100COMNTOOLS%`

この環境変数に格納されているパスの `Tools` 部分を `IDE` に置換すると、devenv.com のあるフォルダー名となる。

Visual Studio 2017～2019
========================

環境変数は廃止されている。そのかわり 2017～2019　では

```
C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe
```

というコマンドがある。これの出力から devenv.exe の場所が特定できる。

```
$ vswhere | findstr productPath
productPath: C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\devenv.exe
```

* devenv.exe は devenv.com と基本機能は同じだが、コンソールに何も出力されない。コンソール出力を見たいので、拡張子を .com に変えておく。
* 32bit Windows では `C:\Program Files (x86)` は存在しない。また、Program Files が C: ドライブ固定とみなすべきではない。vswhere.exe は環境変数 %ProgramFiles% と %ProgramFiles(x86)% 双方の下の `Microsoft Visual Studio\Installer` から探すようにする。

その上で

* 最新のみ → `vswhere -latest`
* 2019 → `vswhere -version [16.0,17.0)`
* 2017 → `vswhere -version [15.0,16.0)`

とすれば Visual Studio のバージョンを指定できる

以上を元にして、我々が呼び出した適切な Visual Studio の devenv.com のフルパスを検出することがほぼ出来るようになる。

検索を自動化して、適切な devenv.com を呼び出すクライアントを書いた
===========================================================

上記の検索処理はバッチでも書けなくもないが、結構たいへんなので、Go言語で自動化した。

* [Visual Studio のコマンドラインクライアントのラッパー vf1s を作った - 標準愚痴出力](http://zetamatta.hatenablog.com/entry/2019/06/22/160031)
* [zetamatta/vf1s: vf1s - Visual Studio Commandline Client](https://github.com/zetamatta/vf1s)

`vf1s -a` で、カレントディレクトリにあるソリューションファイルで、適当な Visual Studio で、全部のコンフィギュレーションをビルドする。またオプション全部指定でいくなら `vf1s.exe -2019 -c "Release|x86" -re hoge.sln` … Visual Studio 2019 指定で、 hoge.sln というソリューションの `Release|x86` というコンフィギュレーションをリビルドするといったことができる。

よろしければ、ご利用いただきたい。

以上

