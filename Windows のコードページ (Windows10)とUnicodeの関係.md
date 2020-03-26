---
title: Windows のコードページ (Windows10)とUnicodeの関係
tags: Windows unicode
author: zetamatta
slide: false
---
Windows では API が２種類ある。

- ANSI系API
   - API名が「A」で終わっている
   - 文字列が MBCS (Multi Byte Charactor System)。１文字が複数バイト
   - 英語＋もう一種類の言語という構成。もう一種類の言語はモードで切り替える（コードページ）
- Unicode系API
   - API名が「W」で終わっている
   - 文字列が W (Wide Character)：UTF16 で表現する
   - 全ての言語の文字をモード切替なく表現できる
   - 昔は１文字がかならず２バイト固定だったが、今は４バイト以上の時もある

Windows で UTF8 を扱おうとすると、２つのアプローチがある。

- 言語のライブラリ層で、UTF8 を UTF16 に変換して、Unicode系APIを使う
     - Cygwin、Go言語などがこの方法をとっている
     - アプリケーションはコードページを気にしなくてよくなる
     - ただし、CMD.EXE がサポートしていない（バッチファイルを UTF16 で書きたい）
- コードページを 65001 (UTF8) に切り替えて、ANSI系APIを使う
     - 一つのフォントは一つのコードページにしか属せない**（過去形）**
        - 日本語環境で `chcp 65001` などでコードページを切り替えても日本語フォントが利用できない**（過去形）**

**Windows10 の最近のアップデートでは、１フォント１コードページという縛りがなくなった**らしく、「`chcp 65001`」の後も、引き続き、それまで使っていたフォントが利用できるようになった。すばらしい

コードページの設定は三種類ある。

* アクティブなコードページ(定数 `CP_ACP`で指定される)
* スレッドのコードページ(定数 `CP_THREAD_ACP`で指定される)
* コンソールのコードページ (API `GetConsoleCP()` で得られる)

子プロセスで CMD.EXE を呼び出すような場合、その CMD.EXE はスレッドのコードページではなく「コンソールのコードページ」で動作する（参考：[problem : batchfile arg UTF-8 data broken. · Issue #322 · zetamatta/nyagos](https://github.com/zetamatta/nyagos/issues/322)）ので、バッチファイルを生成して実行する場合は注意が必要だ。

以上

