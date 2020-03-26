---
title: n番煎じの powershell 向け shebang !
tags: PowerShell バッチファイル Windows
author: zetamatta
slide: false
---
PowerShell のスクリプトを実行しようとすると、実行ポリシーを一時的に変えるため

```
powershell -ExecutionPolicy RemoteSigned -file HOGE.ps1
```

と長い起動コマンドラインになってしまいがちです。

ということで、`HOGE.ps1` を実行するために、別途 `HOGE.cmd` などのバッチファイルを用意したりしますが、ファイルが増えると、配布する手間、説明する手間（「二つを同じフォルダーに置いて実行してください！」とか）も増え、しかも使ってもらえなくなったりして、なかなか難儀です。

そこでバッチファイルの中に PowerShell スクリプトを組み込んで、１ファイル化しようということになります。過去、PowerShell をバッチファイルに組み込む方法は、過去いろいろ記事がありました。

* [PowerShell をバッチファイルに組み込もうとしたけれども（追記：解決した！） - Qiita](https://qiita.com/zetamatta/items/a9b5201b7d8009fad06b) 
* [Windowsでshebangもどき、またはバッチにスクリプトを埋め込む方法 - Qiita](https://qiita.com/snipsnipsnip/items/50e4ca88e3ce3f8cffda)

そんな中、先日見つけた記事：

* [バッチファイルにPowerShellスクリプトを埋め込む - Programming Field](https://pf-j.sakura.ne.jp/program/tips/ps1bat.htm)

これはかなり決定打のように思われました。`echo` がバッチコマンド、PowerShell 両方にある（`echo` = `Write-Output` のエイリアス）ことを利用するなど、よく考えられたものです。ただ、惜しまれるのはバッチとの共用行が長くて、覚えづらいということでしょうか。

そこで、自分なりにアレンジしてみました。

```shebang.cmd
@set "args=%*"
@powershell "iex((@('')*3+(cat '%~f0'|select -skip 3))-join[char]10)"
@exit /b %ERRORLEVEL%

Write-Output $env:args

# vim:set ft=ps1:
```

* プログラム的にバッチファイルの先頭３行を空行に差し替えて、Invoke-Expression(iex)で処理するようにした（３行を差し替えずに、そのまま消してもよかったのだが、エラーの行番号がズレさせたくなかったため）
* 改行を表す「\"`n\"」を [char]10 などに置き換えた（エディターのシンタックスハイライトに優しい）
* 引数は残念ながら `$args[]` ではなく、`$env:args` を使う（とほほ）

しかし、こういうのも、多分、どこかで既出なんでしょうね

