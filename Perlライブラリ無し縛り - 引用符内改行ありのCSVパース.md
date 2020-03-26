---
title: Perlライブラリ無し縛り - 引用符内改行ありのCSVパース
tags: Perl:5
author: zetamatta
slide: false
---
* ダブルクォーテーションに囲まれた文字列中の改行、カンマを区切り文字として扱わない
* ライブラリは使わない。インタプリタの機能のみを使う

```pl
&csvparse( sub{ <DATA> } , sub{ print map("[$_]",@{$_[0]}),"\n"; } );

sub csvparse{
    my ($read,$callback)=@_;
    while( defined(my $line=$read->()) ){
        for(;;){
            $line =~ s/"([^"]+)"/"\a".unpack('h*',$1)."\a"/ge;
            last unless $line =~ /"/ && defined(my $next = $read->());
            $line .= $next;
        }
        chomp $line;
        my @csv = split(/,/,$line);
        s/\a([^\a]+)\a/pack('h*',$1)/ge foreach( @csv );
        $callback->( \@csv );
    }
}
__END__
nihon,go ahaha,ihihi
ahaha,"ihihi , ufufu","ohoho
mumumu
ahaha
gahaha"
XXXXX,"YYYYY","DDDDD,XXXXXX"
```

* CSV解析関数 csvparse の
  * 第一引数は、1行を取得するクロージャを渡す。
  * 第二引数は、分解した CSV を引き渡すクロージャーを渡す。

```
$ perl csv.pl
[nihon][go ahaha][ihihi]
[ahaha][ihihi , ufufu][ohoho
mumumu
ahaha
gahaha]
[XXXXX][YYYYY][DDDDD,XXXXXX]
```

初出：[MHI 5.0 - [Perl] ライブラリ無し縛り - CSVパース](http://nyaos.org/d/index.cgi?p=%282012.10.01%29#p1)

