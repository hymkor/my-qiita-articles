---
title: 【作業メモ】さくらインターネットの標準CGIサーバでGoをビルド【古典的レンタル鯖】
tags: Go FreeBSD
author: zetamatta
slide: false
---
Sakura Internet の FreeBSD の標準サーバで golang を導入しようとしたが、どういうわけか

```
$ wget https://storage.googleapis.com/golang/go1.7.4.freebsd-md64.tar.gz
$ tar zxvf go1.7.4.freebsd-amd64.tar.gz
$ ./go/bin/go version
-bash: ./go/bin/go: cannot execute binary file: 無効な実行形式です
```

などと怒られてしまう。一応、OSはサポート範囲に入っているのだが、root権限の無い、借り物シェルアカウントなので、いろいろ制約があるのだろう。どうも動的リンクライブラリが見つからない状態になっているように思えた。というわけで、ソースからビルドしてみた。

* 問題のサーバでは Go 1.4.3 はバイナリパッケージでも動作した。この Go 1.4.3 バイナリで Go 1.7.4 ソースをビルドすることにする

* ライブラリのビルド中、`run.bash: line 8: /home/xxxxx/go/bin/go: cannot execute binary file: 無効な実行形式です` などというエラーがビルド中のバイナリ実行中に出た。バイナリパッケージを使った時と同様、どうも動的リンクがうまくいっていない可能性が高い。
    * `export CGO_ENABLED=0` で cgo を使わないような設定にすると、静的リンクを強制できる。それで当エラーは解消した。
    * 個人的には cgo は使わない方針にしているので、これで問題ないが、cgo を使いつつも、静的リンクを強制するオプションもあるらしい（調べていない）
* テスト中、`fork/exec /tmp/go-build289658699/archive/tar/_test/tar.test: permission denied` というエラーが出ることがある。ユーザ権限でビルドしているのだが、/tmp 以下にはファイルは置くことはできても実行はできないらしい。`export TMPDIR=$HOME/tmp` などとして、一時フォルダーを自前のディレクトリに変更すると、エラーは解消した。

一応、以上の操作で、最新の Go 1.7.4 をビルドすることができた。

参考
====

- [RT FreeBSDにGoogleのプログラミング言語Goをインストール - willowletの日記 ](http://d.hatena.ne.jp/willowlet/20091129/1259517856)
- [golangで書いたアプリケーションのstatic link化 - okzkメモ ](http://okzk.hatenablog.com/entry/2016/08/03/234738)

