---
title: VisualStudio2015の不具合でDataGridViewの列追加画面が化けた時は一時的に言語パックを英語にすればいいよ
tags: VisualStudio VisualStudio2015 .NET .NETFramework
author: zetamatta
slide: false
---

![image.png](https://qiita-image-store.s3.amazonaws.com/0/29454/cf95f5ef-b48c-8721-266c-4095ad896e87.png)

**バグってる！**

ぐぐった
========

* [DataGridViewタスクの「列の追加」の画面崩れ](https://social.msdn.microsoft.com/Forums/security/ja-JP/16f031c8-749e-4bcb-8abd-6b6a64146555/datagridview?forum=csharpgeneralja)

MSDN の記事はすぐ行方不明になるので、要点を整理すると

* 日本語の言語パックが壊れているらしい
* 英語の言語パックを導入してから、英語の状態で列を追加すればよい

最新のサービスパックで直っているかもしれないけど、念のため

手順
====

1. 英語の言語パックのダウンロード
   * VisualStudioのメニューの「ツール」→「オプション」→「環境」→「国際対応の設定」→「追加の言語を取得する」で、ブラウザでダウンロード画面が出る。
   * ダウンロードした vs_langpack.exe を実行する
2. 英語環境にする
   * 「ツール」→「オプション」→「環境」→「国際対応の設定」→「English」
   * VisualStudio を再起動

![image.png](https://qiita-image-store.s3.amazonaws.com/0/29454/68073aa6-78e6-99e4-6106-72123360919f.png)

バッチリだぜ！

