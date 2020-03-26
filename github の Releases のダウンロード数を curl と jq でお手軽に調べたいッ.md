---
title: github の Releases のダウンロード数を curl と jq でお手軽に調べたいッ
tags: GitHub curl jq JSON
author: zetamatta
slide: false
---
次のような API があるようだ。

* [Releases | GitHub Developer Guide](https://developer.github.com/v3/repos/releases/) → List releases for a repositor

「`GET /repos/:owner/:repo/releases`」の意味が分かりにくいが、「`:owner`」と「`:repo`」を、レポジトリのオーナー名とレポジトリ名に置き換えればよいようだ。つまり、https://github.com/zetamatta/nyagos の情報を見たい時は https://api.github.com/repos/zetamatta/nyagos/releases を開けばよろしい。ブラウザでも大丈夫で、別に認証も要らないようだ。

得られるデータは JSON なので、これを加工するツールが必要だ。自前で作ってもよいが、お手軽に [jq](https://stedolan.github.io/jq/) とやらを使ってみよう。初めて使うので、コマンドがよく分からないが、[jq コマンドを使う日常のご紹介 - Qiita](https://qiita.com/takeshinoda@github/items/2dec7a72930ec1f658af) やら、[jq Manual (development version)](https://stedolan.github.io/jq/manual/)などを見て、試行錯誤してみた結果、次の１行で期待する出力を得ることができだ。

```
curl https://api.github.com/repos/zetamatta/nyagos/releases | jq-win64.exe ".[].assets[] | .name, .download_count"
```

```
"nyagos-4.3.1_3-386.zip"
98
"nyagos-4.3.1_3-amd64.zip"
229
"nyagos-4.3.1_2-386.zip"
65
"nyagos-4.3.1_2-amd64.zip"
86
"nyagos-4.3.1_1-386.zip"
9
"nyagos-4.3.1_1-amd64.zip"
36
：以下省略
```

アクティブユーザ 200人くらいか。やっぱり、もう 64bit版をダウンロードするユーザの方が多いんだなぁ。

これが知りたかっただけだけど、ついでに jq の使い方が分かったので、得たものは大きい

