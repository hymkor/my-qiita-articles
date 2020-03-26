---
title: バッチファイルで、ネットワークドライブ上のパスをUNCパスへ変換
tags: batch
author: zetamatta
slide: false
---
# ソース

**getunc.cmd :**

```bat
@echo off
if "%1" == "" exit /b 1
SETLOCAL
SET "D_=%~d1"
SET "P_=%~pnx1"
for /F "tokens=1,2,3" %%I in ('net use ^| findstr /I %D_%') do if "%%I" == "OK" (set "UNC=%%K" ) else ( set "UNC=%%J" )
if "%UNC%" == "" (
    echo %~dpnx1
) else (
    echo %UNC%%~pnx1
)
```

「net use」では、通常のネットワークドライブは１項目目が「OK」になるが、VMwareの共有フォルダーのように空白になる場合もあるので困ったものだ

# 使用例

```
[Z:Share]
$ net use
新しい接続は記憶されます。


ステータス  ローカル名 リモート名                ネットワーク名

-------------------------------------------------------------------------------
OK           Z:        \\127.0.0.1\c             Microsoft Windows Network
コマンドは正常に終了しました。

[Z:Share]
$ pwd
Z:\opt\Share
[Z:Share]
$ getunc .
\\127.0.0.1\c\opt\Share
```

# 他のバッチファイルから使う

```
$ type foo.cmd
@echo off
for /F %%I in ('getunc "%HOME%\BIN"') do set "BIN=%%I"
echo %BIN%
[Z:SkyDrive]
$ foo
\\Vboxsvr\c_drive\hymkor\SkyDrive\bin
```

