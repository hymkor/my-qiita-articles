---
title: Microsoft.Jet.OLEDB.4.0でCSVに実数をSQLでInsertすると小数点以下２桁未満が切られる
tags: .NET CSV
author: zetamatta
slide: false
---
一般には `schema.ini` というファイルを CSVファイル群が格納されているディレクトリに置きます。そこで CSVファイルの各カラムの型を Single とか Double とか明示します。
（schema.ini については、そのまんまググれば情報が出てくるかと思います）

が、今回 Single のカラムに「`1.23456`」という値を INSERT 文で入れても「`1.23`」で切られるという現象がありました。

原因は `schema.ini` の各テーブルの設定に `NumberDigits` という項目を明示していなかったためでした。

[テキスト データ ソース ドライバーを初期化する](https://msdn.microsoft.com/ja-jp/library/office/ff834391.aspx?f=255&MSPPError=-2147217396)
**NumberDigits**

    Indicates the number of decimal digits in the fractional portion of a number. 
    If this entry is absent, the default value in the Windows Control Panel is used.

    数値の少数部分の十進数の数を示す。この項目が空の場合はコントロールパネルの値が使われる

`NumberDigits` を定義していないと、

	OS のコントロールパネル
	→ 地域と言語
	→ 追加の設定
	→ 小数点以下の桁数

の値が `NumberDigits` として採用されるようです。Windows7 , 8.1 ではこの値が 2 になっており、手動で9などに変更すると連動して、CSVに出力される桁数が増えました。今回の対応としては、OSの設定で左右されるのは問題なので、各テーブルごとの NumberDigits を全て9にすることにしました。

正直、CSVの入出力は基本SQLでやるものではありませんね。本件はDBを使ったシステムをDB-Less にする必要があったため、てっとり早い方法をとりましたが。

以上、誰かがググった時のために（日本語情報、まるでなかったので）

