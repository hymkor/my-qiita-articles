---
title: テスト用のWindows10の仮想環境（寿命90日）の作成と日本語化
tags: Windows10
author: zetamatta
slide: false
---
（１）90日限定の試用版 Windows10 をダウンロード
===============================

* https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/

OVAイメージ(4GB)をダウンロードできる。
このイメージは一度立ち上がると、90日後に使えなくなるが、立ち上げる前の状態のスナップショットに戻せば、何回でも使える（らしい：未検証）。
ただ、WindowsUpdate の手間暇を考えると、都度、ダウンロードしなおした方がよいかもしれない。

（２）VirtualBox をダウンロード＆インストール
============================

* https://www.virtualbox.org/wiki/Downloads

わたしの環境では VirtualBox が「インストーラー サービスにアクセスできませんでした」というエラーが発生して、インストールが止まってしまった。この場合の対処法は Microsoft のサポートサイトに記載されている。

* [Windows 7 または Windows Vista でプログラムをインストールしようとすると、"Windows インストーラー サービスにアクセスできませんでした" エラーが発生する](https://support.microsoft.com/ja-jp/help/2642495/-the-windows-installer-service-could-not-be-accessed-error-when-you-try-to-install-a-program-in-windows-7-or-windows-vista)

（３）VirtualBox の起動＆イメージをスナップショット
=============================

1. VirtualBox を起動 → ファイル(F) → 仮想アプライアンスのインポートより、OVFイメージをインポート
2. 「MS Edge - Win10_preview」というマシンが出来るので、スナップショット(S)→ 📷 カメラアイコン(Ctrl-Shift-S)で、スナップショットを取る

（４）イメージの起動と WindowsUpdate
========================

1. VirtualBox をインストールした後、一度もホストOSをリブートしていない場合は、一度リブートする（これをしないとゲスト起動が失敗する）
2. ゲストの「MS Edge - Win10_preview」を起動する
3. ユーザは自動ログインされる。壁紙に全情報が書いてある
    * ユーザ名「IEUser」
    * パスワードは「Passw0rd!」
        * 最初のPは大文字、途中の0は数字のゼロ、末尾には感嘆符がついている
4. とりあえず、WindowsUpdate しよう
     * ネットワークがオフラインになっているというエラーがでることがある。
    実際にネットワークの設定が出来ていない場合もあれば、起動直後だからまだネットワークのサービスが起動しきっていないだけのこともある。
    一度くらいためしに「リトライ」してみよう。
5. 待つ（すごい時間かかる）

***＜このあたりでスナップショットを取っておいた方がよいと思われる＞***

（５）日本語化
=========

1.  「Setting」
    * →「Time and language」
    * →「Date & Time」
    * → Time Zone を「(UTC+9:00) Osaka, Sapporo Tokyo」にする
2. 「Region & language」
    * Countory & region」を「Japan」
    * 「Language」で「Add a language」「日本語」
    * 「日本語」をクリック
        * 「Set Default」
        * 「Language Options」→「Download」
        * 「Hardware keyboard layout」→「Change layout」→「Japanese keyboard(106/109 keys)」& Sign out

（６）コマンドプロンプトのchcp 932 対応
=========================

* 「Windowsアイコン」を右クリック 
* →「コントロールパネル（P）」
* →「時計・言語・および地域」
* →「日付、時刻、または数値の形式の変更」
* →「管理」タブ
* →「Unicode対応ではないプログラムの言語」
* →「システムのロケールの変更」
* →「日本語（日本）」
* →「今すぐ再起動」

***＜ここまでが結構な手間なのでスナップショットを取っておいた方がよいと思われる＞***

以上 （http://zetamatta.hatenablog.com/entry/2017/05/28/233643 より転載）

