---
title: nyagosスクリプト解説 - svn のサブコマンドを勝手に拡張する
tags: nyagos
author: zetamatta
slide: false
---
そう、nyagos ならね！

```lua:%USERPROFILE%\.nyagos
nyagos.alias.svn = function(args)
    if args[1] == "amdc" then
        return "svn.exe status | findstr ^[AMDC]"
    end
    args[0] = "svn.exe"
    if args[1] == "exclude" then
        args[1] = "update"
        table.insert(args,2,"--set-depth=exclude")
    elseif args[1] == "emptyup" then
        args[1] = "update"
        table.insert(args,2,"--depth=empty")
    elseif args[1] == "emptyadd" then
        args[1] = "add"
        table.insert(args,2,"--depth=empty")
    end
    return args
end
```

- `svn amdc` … `svn status | findstr ^[AMDC]` と同じ。つまり、改変があったファイルの一覧のみを表示して、？なファイルは表示しない
- `svn exclude` … `svn update` で出て来たファイルを出てこなかったことにする（レポジトリからは消さないが、ローカルからは消す）
- `svn emptyup` … ディレクトリを `svn update` するが、その中のファイルは`svn update`しない
- `svn emptyadd` … ディレクトリを `svn add` するが、その中のファイルは`svn add`しない

### 解説

- エイリアス定義はいろいろな書き方が出来るが、4.0.9 では「`nyagos.alias.エイリアス名`」に関数なり文字列なりを直接代入できるようになった（nyagos.exe 自体に組み込まれた機能）
- エイリアス関数では関数自体で処理を記述せずとも、戻り値に新たなコマンドラインを返すという方法もとれるようになった。
       - 値としては、文字列、もしくはテーブルのどちらでもよい。テーブルの場合、実行ファイル名はテーブルの要素０でも１のどちらでもよい。これはエイリアス関数の引数のテーブルが１から始まり、かつエイリアス名そのものが含まれていないので、テーブルをシフトする手間がかからないように…という配慮だが、まぁ、**いきあたりばったり感は否定できない**
- 戻り値に含めるコマンド名は、元のコマンド名と同じにすると、無限ループになりかねないので、ここでは「svn.exe」と拡張子を付けて回避している。
       - 戻り値ではなく、`nyagos.exec` / `nyagos.eval` を使う場合でも無限ループが発生しないよう同様の工夫が必要だが、これとは別に `nyagos.rawexec` / `nyagos.raweval` という関数を使うという方法もある。これらはエイリアスとか内蔵コマンドと解釈されない。Lua側の標準関数にも os.execute というものがあるが、こちらは文字列を UTF8 ではなく、現在のコードページの文字列として解釈されるので、文字化けが発生する場合がある

### 追記（gogit.lua について）

4.0.9_6 には「[gogit.lua](https://github.com/zetamatta/nyagos/blob/master/catalog.d/gogit.lua)」というスクリプトを catalog.d\ に置いてある（catalog.d\ は標準で読み込まれない任意スクリプトを置くディレクトリ）。これはサブコマンド名を見て、「go」と「git」を書き間違えを補正するというフィルターである。こちらは作りがよりシンプルなので、よろしければ参考にしていただきたい。

