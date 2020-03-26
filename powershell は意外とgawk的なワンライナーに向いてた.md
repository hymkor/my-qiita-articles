---
title: powershell は意外とgawk的なワンライナーに向いてた
tags: PowerShell バッチファイル gawk Windows
author: zetamatta
slide: false
---
とあるツールをバッチファイル + gawk で書いてたけど、いざ他人に渡そうとすると gawk がないだろうから、Windows 標準コマンド固めにしたかったー

# ipconfig の結果から最初の IPv4 のアドレスをゲットする

```
ipconfig | gawk "/IPv4/{ print $NF ; exit}"
```

IPv4 を含む行を抽出するのと、最後の項目を抜き出すのは `for /F` と `findstr` で代用できるが、意外と最初のものだけを出すというのが厳しい。まじめに JScript あたりでスクリプト書く必要あるかと思ったが、PowerShell ならワンライナーで済ませられた。

```
powershell -Command "ipconfig | ?{ $_ -match '^ *IPv4'} | Select-Object -first 1 | %{ $_.Split()[-1] }"
```

* PowerShell はシングルクォートを文字列を囲む記号に使えるのでワンライナーにも適していた

# あらかじめ、ファイルに落としておいた ipconfig の結果を同様に扱う

```
powershell -Command "Get-Content $env:userprofile'\Share\etc\Hayama-pc.txt' | ?{ $_ -match '^ *IPv4'} | Select-Object -first 1 | %%{ $_.Split()[-1] }"
```

* `%USERPROFILE%\Share\Etc\Hayama-PC.txt` に、あらかじめ ipconfig の結果が入っている。目的としては他の PC の(DHCPで割り当てられた）IP アドレスを OneDrive 経由でゲットしたかった
* 環境変数をゲットするのは `$env:userprofile`で、文字列連結は`+`のはずだが、なぜか `+` を書くと、結果の文字列に `+` が入ってしまうので、そのまま横にならべた。なんでじゃろね
* バッチファイル中のものを引用したので「`%`」が「`%%`」になってます

# ゲットした IP アドレスを環境変数に格納しなおす

```
for /F "usebackq" %%I in (`powershell -Command "Get-Content $env:userprofile'\Share\etc\Hayama-pc.txt' | ?{ $_ -match '^ *IPv4'} | Select-Object -first 1 | %%{ $_.Split()[-1] }"`) do set "REMOTEIP=%%I"
```

* `for /F` でコマンド出力を取り込めるが、二重引用符も一重引用符も powershell で使ってしまっている。そんな時、`for /F "usebackq"` を使えば、in 句内で逆引用符が使える
* 結果の IP アドレスは `%REMOTEIP%` という環境変数に取り込む

この IP　アドレスを `ttermpro.exe` (TeraTerm) の ssh フォワーディングのためのパラメータに渡してだね…あ、だれか来たようだ。

もっと簡単にやる方法もありそうな気がするが、それはコメントを待つことにしよう







