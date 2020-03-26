---
title: NYAGOS なら Unicode ファイル名にも自信が持てます
tags: Go:1.3 Lua:5.2 nyagos
author: zetamatta
slide: false
---
# NYAGOS とは

Go 言語で **作成中** の Windows向けの Unicode 対応 UNIX風コマンドラインシェルです。知る人ぞ知る、NYAOS シリーズのバージョン「4」に相当します（今年でなんと18年目となります）

そして、ぼちぼちと、このNYAGOSも「使えなくもないかなー」レベルにまでなってきたので、ここいらで宣伝記事を書かせていただきます。

> ちなみに「1」は OS/2 向け、「2」は DOS・Windows・OS/2向け、「3」は「2」をベースに Windows+OS/2 向けに絞って、Lua による拡張機能を付けたもの。だが、基本的に Double Byte Charactor System が大前提だったため、Unicodeに対応するには「また」スクラッチから作り直す必要があった

* ソース：https://github.com/zetamatta/nyagos
* バイナリ：http://www.nyaos.org/index.cgi?p=NYAGOS
* ドキュメント：<br /><del>https://github.com/zetamatta/nyagos/blob/master/nyagos_ja.mkd</del><br /><ins>https://github.com/zetamatta/nyagos/tree/master/Doc/</ins>

# 特徴

* NYAOS シリーズ伝統のUNIX風コマンドラインを Windows に提供するネイティブシェル
   * UNIX風１行入力編集
   * エイリアス・ヒストリ
   * 内蔵カラーls、カラープロンプト
* 今回、Go 1.3 にて開発
   * 従来の NYAOS より開発効率が大幅にアップした。
   * 外部のライブラリも積極的に活用
* 設定とカスタマイズは Lua 5.2 で記述
   * まだ、alias と setenv しかないけど。
   * ポリシーとしては go で作成する本体はできるだけ必要最小限な構成とし、Lua で機能を作り込む / 改造できるようにする。
* Unicode対応
   * 内蔵 ls にて、Unicodeファイル名を表示できる
   * 「`echo ☑ > ☑`」などというコマンドも正常に機能する。
        * ただし、ファイル「`☑`」の中身は UTF8 の`☑`
* 従来の NYAOS だと、環境変数設定バッチをうまく実行できないという問題があり、`cmdsource.lua` という外部Luaライブラリで解決していた。今回、本体側で対応し、`source バッチファイル名` でそういったバッチファイルによる環境変数変更を取り込めるようになった。
* ソースは Github にて「BSDライセンス」にて公開中。


# 残念なお知らせ

## 悪いな、のび太。 NYAGOS はまだ Shift-JIS には対応してないんだ。

これから実装する予定の逆クォートとか、文字コード変換どうするんだ？更に、Lua の文字列は基本バイト列だろ…

でも、chcp とかしなくてもいいんだぜ（go言語が UTF16 のAPIを使ってくれているおかげ）。

## 当初は mruby をカスタマイズ言語にしたかったんだ。

でもな、mruby には LuaBinaries みたいな Windows 向けのバイナリパッケージないだろ…（DLLを作るパッケージはあったが、最新mruby に追随できておらず、goの仕様上、一筋縄で組み込めない）。本当は ShiftJIS と UTF8 の架け橋をmrubyに担って欲しかったんだ…。

## まだ、バージョン 1 じゃない

機能も NYAOS-3000 に全く及ばない。

* 上にも書いたが、逆クォートも「まだ」ない。
* 一行入力は「まだ」完全じゃない。
* Lua で（カレントディレクトリや環境変数を操作できるような）内蔵コマンドも「まだ」作れない。

今のところ、最新ソースと、時々のスナップショットのバイナリを公開しているという状況で、ただちに NYAOS-3000 からリプレースできるようなものにはなっていません。今後に乞うご期待というところです。ということで、応援よろしくお願いいたします。

# その他

## 使用させていただいているライブラリ

* http://github.com/mattn/go-runewidth
* http://github.com/shiena/ansicolor
* http://github.com/atotto/clipboard

さらに @mattn_jp さんと @nocd5 さんには度々アドバイスをいただいております。この場を借りてお礼申し上げます。

