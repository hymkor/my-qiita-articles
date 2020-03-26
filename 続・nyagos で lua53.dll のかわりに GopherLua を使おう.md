---
title: 続・nyagos で lua53.dll のかわりに GopherLua を使おう
tags: Go Lua GopherLua nyagos
author: zetamatta
slide: false
---
「[nyagos で lua53.dll のかわりに GopherLua を使おう - Qiita](https://qiita.com/zetamatta/items/112484eb7fdae87830a0)」の続き：

β2 を出しました。

* [Release 4.3_beta2 · zetamatta/nyagos](https://github.com/zetamatta/nyagos/releases/tag/4.3_beta2)

**リポート！**

Luaスクリプトの互換性
===================

GopherLua 起因と思われる不具合はありませんでした。。あったのは Lua のバージョンが 5.3 → 5.1 に変わることによる文法上の差異のみです。

nyagos.d/ 上のスクリプトと、自分の環境上のスクリプトの修正内容は下記のとおり。

* 5.1 には `table.unpack` がない<br />⇒　`unpack` とする（どちらでも動くように `(table.unpack or unpack)` とするのが個人的には好き）<br />【nyagos.d/aliases.lua】
* 5.1 では `load("luaコマンド")` がない<br/>⇒ 変数 `_VERSION` が`"Lua 5.3"` でなければ `loadstring("luaコマンド")` とする<br /> 【nyagos.d/aliases.lua】
* 5.1 にはビット操作演算子(`&`、`|`)がない<br /> ⇒ 自前で関数 `nyagos.bitand`、`nyagos.bitor`を用意した<br /> 【nyagos.d/box.lua】
* 5.1 には `math.tointeger` がない<br/> ⇒ 変数 `_VERSION` が `"Lua 5.3"` でなければ、浮動小数点をそのまま渡す。受け取る利用側の Go 関数の方で値を整数化して使うようにした。<br />【nyagos.d/trash.lua】
* 5.1 には "\xHH" 形式の16進文字列リテラルがない<br/>⇒使わないようにした<br />【テストコード：t/tst_utoa.luaなど】

実際の Lua コード修正内容は

```
cd nyagos.d
git diff 4.2.5_1 .
```

で確認できます。

ユーザの反応
===========

思ったよりも肯定的。Windows で Lua を LuaRocks などからパッケージをダウンロードしてまで使おうという人があまりいないためかもしれません（自分もやったことない）。

その他の影響
===========

* 目論見どおり、appveyor でのビルド失敗率は減った。ダウンロード先は github.com だけになったのが大きい
* zipサイズは 300～400KB 程度、exeファイルは 1.3MB 程度増えた。
* zipファイル作成時に lua53.dll が32bitと64bit取り違えてないか、心配する必要がなった。
* Pure Go 言語になったため、panic が発生した時のスタックトレースが常に信頼できるようになった（気がする）。Go言語どおしなので、おそらく Lua に渡すコールバック関数で、トリッキーなことをしても大丈夫だろう（やってないけど）

ソース（参考）
============

* [`nyagos/gopherSh/*.go`](https://github.com/zetamatta/nyagos/tree/master/gopherSh) … GopherLua 版のメインパッケージ
* [`nyagos/mains/*.go`](https://github.com/zetamatta/nyagos/tree/master/mains) … lua53.dll 版のメインパッケージ
* [`nyagos/ngs/*.go`](https://github.com/zetamatta/nyagos/tree/master/ngs) … Lua が使われないバージョンのメインパッケージ

です。 

* Lua に依存したコードは現在すべて `gopherSh/` か `mains/` 以下に封じ込めている
* 将来的には `mains/` を削除して `gopherSh/` を `mains/` にリネームするかもしれませんが、当面は維持

処理系変更時の工夫
=================

nyagos開発の当初からやっていたこと
-------------------------------

* lua を使うコードは、メインパッケージ(mains/)だけに封じ込める（サブパッケージで lua 機能を使う時は、メインパッケージから依存部分をパラメータ等で与えるよう、徹底する（「依存性抽出」という表現でいいのかな？）
* エイリアスや一行入力キーに Lua 関数をアサインする時も、Go の interface オブジェクトをテーブルに設定するようにし、決して Lua オブジェクトを関数プロトタイプに含めたものを直接使わないようにした。

しかしながら、この結果として**メインパッケージが全ての混沌を引き受け、肥大化した**わけです。以下は、**この肥大化したメインパッケージから、いかに Lua 成分を入れ替えるか**という戦いが始まります。

コールバック関数を両方で使えるように
---------------------------------

Lua へ渡すGoのコールバック関数の仕様は、

* 自前の lua53.dll ラッパー：`func(lua.Lua) int` ※
* GopherLua ：`func(*lua.LState) int`

となっています。（※lua.Lua は、C言語の lua_State* の値(uintptr)を type したもの)

一つ一つ書き換えてもよいのですが、あまりに量が多いため「書き換え中のビルドできない過渡期」が長くなりすぎてしまうおそれがありました（**たぶん、途中で絶対心折れる**）。そこで処理系に依存しない

* `func([]interface{})[]interface{}`

という形式を、Lua へ渡すコールバック関数の標準形式とし、これを各 Lua 処理系に適した形に変換する関数：

* `func lua2cmd(f func([]interface{}) []interface{}) func(lua.Lua) int`
* `func lua2cmd(f func([]interface{}) []interface{}) func(*lua.LState) int`

を用意しました。（実際には、これだけだとリダイレクトされた標準入出力(`io.Writer` / `io.Reader` )の引き渡しができないので、もう一種類形式も用意）

こうした工夫の結果、`func([]interface{})[]interface{}`という形式への切り替えは 2017年4月に着手した※ものの、それは他の修正作業を妨げることなく「裏で」こっそり進めることができました。

（※追記：多分、本格的な裏着手は 2017年8月頃から。`interface{}` といちいち書いてられなかったので、Go 1.9 の型エイリアスで「`type any_t = interface{}`」で短く書けるようになってからのはず）

Lua 処理系を利用しないバージョン(ngs.exe)を一旦作成
--------------------------------------

appveyor でのビルドエラーが目立つようになってきた 4.2.5 リリース以降から、本腰を上げて、lua53.dll 脱却に着手しました。その第一ステップとして、メインパッケージを二つに分け、lua53.dll を使わないバージョンをまず作る事にしました。

1. mains/ フォルダーを ngs/ というフォルダーにまるまるコピーして、lua に依存する箇所をカットしてゆく。
2. カットするには大きい（明らかに後から作り直すのはしんどい）箇所は、別途フォルダー(`functions/`,`framework/`)を用意し、Lua へ依存する箇所をパラメータ等で引き渡す方式に変更してゆく
3. 最終的に ngs/ でビルドしたバージョンが lua53.dll なしでエラーなく動作すれば成功

実際は、ここまでスムーズにやっているわけではなく、フォルダー構成を右往左往しながら変えてたりします。この作業では、**Lua に依存したソースを mains パッケージからさらに絞る**意味があったと思います。

GopherLua 組み込み開始
---------------------

ngs.exe が出来た時点で「**もう無理に Lua を差し替えなくても、Lisp とか別の言語系を組み込んでもよいかな**」などという浮気心も出始めましたが、

* Goに移植された有名な言語処理系で、適当なものが浮かばなかった
* なんだかんだ言って lua53.dll と GopherLua は引数引き渡しの形式がスタックインターフェイスと同じなので、理論上 100% の差し替えが可能
* GopherLua は[expect for Command Prompt](https://github.com/zetamatta/expect)というツールで、一応、経験もあった

という理由で、結局、当初の目標である GopherLua へ立ち返ります。

Luaなしバージョン ngs/ 以下を、gopherSh/ (当初は gopher/) というフォルダーへ複写しました。そして、ngs/ と mains/ の差分を見ながら、少しづつ移植してゆきます。

1. 最初の目標は、とりあえず `.nyagos` や `nyagos.d/` 以下を読むようにすること（エラー上等）
2. 次の目標は、`func([]interface{})[]interface{}` という標準形式にしてあるコールバック関数群(`nyagos.*****`) を使えるようにすること（具体的には GopherLua 版の引数変換関数 ` lua2cmd` を実装する）
3. lua関数によるのエイリアスの実装
4. ここまで来たら、GopherLua についての知識も十分付き、軌道にも乗ったので、気付いたところから以下を実行してゆく
      * `func([]interface{})[]interface{}`形式に直せていないコールバック関数の個別移植
      * コマンドラインフィルター(`nyagos.argsfilter`,`nyagos.filter`)の類の移植
      * 一行入力のキーへの Lua 関数の割り当て
      * えとせとらえとせとら

ここまでくると、結構ノリノリでコードを書いているので、障害らしい障害はほとんどありませんでした。で、nyagos.d の読み込みエラーも消えたところで、β1 のリリースというわけです。

最後に
======

しかし、まぁ、われながら、よくやったよ

以上

