---
title: git 付属のPerlやunzipとかを使いたいけどsort,find,lnとかは要らない
tags: nyagos msysGit
author: zetamatta
slide: false
---
C:\Program Files\Git\bin を %PATH% に加えると、msysGit についてくる UNIX系コマンドが入ってくる。Perl とか unzip はは嬉しいが、bash ではなくコマンドプロンプトから使う場合など、sort,find,ln などあまり使いたくないコマンドまでたどれてしまう。できれば、必要な奴だけ、ピンポイントで使いたい。

最初 mklink で、git\bin にある perl.exe のハードリンクを、他の%PATH%ディレクトリに作ろうとしたが、DLL を見失うせいか、だめだったー

ということで、nyagos の alias で、必要なコマンドだけをフルパスに変換する暴挙に走った。

```Lua:.nyagos
do
    local gitbin = nyagos.which("git.exe"):gsub("[^\\]+",function(s)
        if s == "cmd" then
            return "bin"
        end
    end):gsub("\\[^\\]+$","")

    for _,name in ipairs{ "perl","unzip" } do
        nyagos.alias[ name ]= string.format('"%s\\%s"',gitbin,name)
    end
end
```

結果

```
$ alias | gawk '/perl/ || /unzip/'
unzip="C:\Program Files (x86)\Git\bin\unzip"
perl="C:\Program Files (x86)\Git\bin\perl"
```

お分かりいただけたであろうか

