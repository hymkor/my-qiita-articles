---
title: NYAGOSの小TIPS集
tags: nyagos
author: zetamatta
slide: false
---
NYAGOS 4.2.0 以降が対象です。

ファイルはゴミ箱へ
=================

    trash ファイル名

でファイルがいきなり削除ではなく、ゴミ箱に入ります。実体は Lua スクリプトで、COM経由でゴミ箱へファイルを移動しています。(`nyagos.d/trash.lua`）

UTF8 変換に type 文
===================

内蔵 type 文は下記の拡張されています。

* 行単位で UTF8 変換
    * テキストが UTF8 として妥当であれば UTF8 と判断、不適当なら現在のコードページの文字種類と判断（つまり、混在データでも大丈夫）
* 標準入力からも読み込み可能

これを利用すると

    git show . | type | gvim - &

などということが可能になります。

* Windows で git やってると、git の日本語メッセージと、ソースのテキストで文字コードが違っていて文字化けが回避できないことがあるが、内蔵 type 文は行単位で判別しているので、ほぼ読める内容になる
* nkf (Network Kanji Filter）の文字コード判別ロジックに比べると、**「エラーだったら UTF8 と違う(ANSI=現在のコードページ)」という実に雑なUTF8判定**だが、案外十分
* NYAOS-3000 では内蔵コマンドをパイプラインの最初以外のコマンドに使えなかった。NYAGOS ではどこでも内蔵コマンド(Lua含む)が使える

深いディレクトリ移動はショートカットで
===================================

**いちいち `cd ~/go/src/github.com/zetamatta/nyagos` なんて長いパスを打ってられるか！**

そこでショートカットですよ。

    lnk ~\go\src\github.com\zetamatta\nyagos .

と一度ショートカットを作成しておけば、以後は

    cd nyagos.lnk

だけでいいです。

* ジャンクションとかシンボリックリンクでもいいんですが、見かけのカレントディレクトリの位置まで変わってしまうので（親ディレクトリがホームディレクトリになってしまう）、単にディレクトリのタイプ数を節約するだけなら、ショートカットで十分でしょう。
* lnk はショートカットを作る内蔵コマンドですが、別にエクスプローラで作ったショートカットでも構いません（同じものです）

git push 漏れはないかな？
=======================

%APPDATA%\NYAOS_ORG\nyagos.history に、タイプしたコマンド、日時、カレントディレクトリ全部が TSV 形式で記録されるようになりました。これを検索する PowerShell を書いてみましょう。

```todaygit.ps1
$done = @{}
Join-Path $env:appdata 'NYAOS_ORG\nyagos.history' |
    %{ Get-Content $_ -Encoding utf8 } |
    ?{ $_ -match '^[g]it commit' } |
    %{
        $private:tmp=($_ -split "`t")
        # $private:from = (Get-Date).AddDays(-3).ToString("yyyy-MM-dd")
        $private:dir=$tmp[1] 
        if( (Test-Path $dir) -and (-not $done.ContainsKey($dir)) ){
            Write-Host $dir
            pushd $dir
            git status | ?{ $_ -match "^Your branch is" } | %{ "   "+$_ }
            popd
            Write-Host
            $done[ $dir ] = 1
        }
    }
```

```console
$ todaygit.ps1
C:\Users\hymko\go\src\github.com\zetamatta\nyagos
   Your branch is up-to-date with 'origin/second'.

C:\Users\hymko\go\src\github.com\zetamatta\nyagos.wiki
   Your branch is up-to-date with 'origin/master'.

C:\Users\hymko\go\src\github.com\zetamatta\experimental\zero
   Your branch is up-to-date with 'origin/master'.

C:\Users\hymko\Share\bin
   Your branch is up-to-date with 'origin/master'.
```

* 普通、PowerShell スクリプトを実行する時は「`PowerShell -ExecutionPolicy RemoteSigned -file` などとセキュリティーオプションをつけなくてはいけません。同オプションをデフォルト化することもできますが、OSユーザ全体にその設定が及んでしまいます。nyagos の場合は、デフォルトで .ps1 拡張子の時に「`suffix ps1="powershell -ExecutionPolicy RemoteSigned -file"`」というインタプリタ名を追加する設定になっており、nyagos から呼び出す時のみスクリプト実行を許可されることが可能になっています。
* 最初は「その日に行った git commit」のみを対象としていたのですが、月曜朝に土日の commit 漏れをチェックするといった時に困ること、どのみち history には 1000件程度しか記録していないことから、別に日付を限定する必要はないと判断しました。

外部の sudo を使いたい
=====================

sudo コマンドは Windows では一般に管理者権限でコマンドを実行する実装が多い。NYAGOS でも内蔵コマンドとして実装していたが、新しいコンソールを立て上げてしまうので、コンソールが変わらない、他の sudo for Windows を使いたい。

そんな時は

    alias "sudo=sudo.exe"
 
と EXE 版を呼び出すよう、 `~\_nyagos` にエイリアスを書きましょう。

* 不評な内蔵版 sudo は次のバージョンでは削除される予定。`su` や `clone` は存続

以上

