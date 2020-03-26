---
title: ストックしたら負けのVB6アプリの .NET 移行メモ
tags: VisualStudio VB VB.Net
author: zetamatta
slide: false
---
# VB6 → VB.Net のコンバーターは VisualStudio 2010 以降、搭載されなくなった

一旦、次の Visual Studio のいずれかで、古いバージョンの VB.Net にコンバートする必要あり

- Visual Studio .NET (2002)
- Visual Studio .NET 2003
- Visual Studio 2005
- Visual Studio 2008

# Visual Studio 2008 （例）で VB6 ソースをコンバートする

コマンドプロンプトより：

```
$ "c:\Program Files\Microsoft Visual Studio 9.0\VB\VBUpgrade\VBUpgrade.Exe" (プロジェクト).vbp
Microsoft(R) Visual Basic アップグレード ツール バージョン 9.0.20913.0
Copyright(C) Microsoft Corporation.  All rights reserved.
Portions Copyright (C) Artinsoft Corporation

アップグレードしています。....................
例外が発生しました: 参照された コンポーネント を読み込めませんでした:
　　　（中略）
参照されたすべてのコンポーネントを含めて Visual Basic 6.0 をインストールすることを推奨します。アップグレードの前に、アプリケーションがコンパイルおよび実行されることを確認してください。
```

# コンバートエラー解消

* 推奨されたので Visual Studio 6.0 も入れる
* コンポーネントを含む製品をインストールする
* どうしても該当製品の入手が難しい場合、Visual Studio 6.0 で `(プロジェクト).vbp` を開いて、「プロジェクト」→「参照設定」から該当ライブラリファイルを消す

上記を行うことで、VBUpgrade.Exe でのコンバートは出来るようになり、サブディレクトリ OutDir 以下にコンバート結果が生成される

# 最新の Visual Studio へ更にコンバートする

```
$ start OutDir\(プロジェクト).vbproj
```

で開くと、変換ウィザードが起動する。最終的に `(プロジェクト).sln` が出来れば Ok。

# ひたすら文法エラーを解消する

俺たちの戦いはこれからだ！Never End！
（そういって、数百件の文法エラーに立ち向かってゆく主人公）

# (必要に応じて)VB6 のインストールされていない環境に移行する

現状だと、VB6関連の DLL が必要となってしまう。そのため、なるべく早く、VB6が無い環境に移行して、VB6 の DLL に依存していたコードを修正する。
（自分の経験だと、表関連のコントロールを、DataGridView まわりに改めなければいけなくなることが多かった）

# その他（トラブルシューティング）

## フォームのデザイナーが開けないケース

デザイナーで開こうとすると、「**値を Null にすることはできません。パラメーター名: objectType**」などというエラーが出て、「無視して続行する」を行わない限り、フォームデザイナーが開けない場合がある。

原因は全く不明だが、コードの方の `Handles` 句を一旦 `' Handles...` 等とコメントアウトしてからだとデザイナーで開ける。一度、デザイナーで正常に開けると、コメントアウトを元に戻しても、普通に開くことが出来た。どういう理由で直るのか不明

# (2019.10.04) 追記 ＜推奨作業＞

コンバート作業自体は以上で Ok だが、今後メンテナンスする上では以下を実施しておくことを推奨する

* インデントをタブだけにするか、スペースするだけかを統一する
* 'UPGRADE_WARNING の全削除

これを後まわりにすると、`svn blame` `git blame` などで「その行がいつ修正されたかのトレースする」が面倒になる。全ソースが修正される最初のうちに、全ソースに修正が入る以上の対応を行っていた方がよい

# 参考文献

- [Microsoft Visual Studio - Wikipedia](https://ja.wikipedia.org/wiki/Microsoft_Visual_Studio)
- [VB 6 を VS 2012に変換するには？ - Visual Basic | 教えて！goo](http://oshiete.goo.ne.jp/qa/7794035.html)

