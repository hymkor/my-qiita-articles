---
title: PowerShell をバッチファイルに組み込もうとしたけれども（追記：解決した！）
tags: PowerShell:1.0
author: zetamatta
slide: false
---
※ 本文で不明であった原因が後に解決したので、コメントとして追記しました。(2014.10.10)

PowerShell はセキュリティーのため、デフォルトではスクリプトファイルを実行できないようになっている（解除の仕方の理論は知ってる）

自分が使うのにはいいけど、人に配る際、いちいち説明せんといかんのはメンドクサイので、バッチファイルの中に組み込んで、標準入力から実行コードを読み込ませるようにしてみた。
（参考：[Windowsでshebangもどき、またはバッチにスクリプトを埋め込む方法](http://qiita.com/snipsnipsnip/items/50e4ca88e3ce3f8cffda))

```ps1:why.cmd
@more +1 "%~f0" | powershell -Command "-" & exit /b 0
Write-Host 'Step1'
Write-Host 'Step2'
if ( $true ){
    Write-Host 'Step3'
    Write-Host 'Step4'
} else {
    Write-Host 'Step5'
}
Write-Host 'End'
```

```text:実行結果
✏ why.cmd
Step1
Step2
✏
```

if 文以降が何故か実行されない… **なんでじゃろね**

more が悪いのかと思って、tail +2 とかに変えてみたりもしたんだけど、結果は変わらないようだ。あと、これくらいのテスト用スクリプトだと、-Command "～" の中に組み込めるけど、長い奴はそうもいかんしね。別に困ってないけど、困った（どっちだ）

