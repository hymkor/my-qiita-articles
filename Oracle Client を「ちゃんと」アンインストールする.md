---
title: Oracle Client を「ちゃんと」アンインストールする
tags: oracle Windows
author: zetamatta
slide: false
---
Oracle Universal Installer でアンインストールを試みると

![image.png](90359a46-5ff4-456f-8751-27eb5ddf78b3.png)

というメッセージが出る。これにしたがって、削除バッチ `deinstall.bat` を実行する時の注意点は３つ

- 管理者権限でバッチファイル(`deinstall.bat`)実行する
     - 「コマンドプロンプト(管理者)」で実行する
     - 管理者モードにならないと、まったく削除できない
- うっかり c:\app\denki\… ディレクトリをカレントディレクトリにしない
     - カレントディレクトリにするとフォルダー削除に失敗する。実害はないが、エラー終了するので気持ち悪い
- ODBC の設定などは、アンインストールする前に削除しておく
     - アンインストール後は ODBC の設定の削除は出来なくなる

以上

