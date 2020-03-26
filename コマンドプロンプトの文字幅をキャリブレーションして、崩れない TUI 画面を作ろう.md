---
title: コマンドプロンプトの文字幅をキャリブレーションして、崩れない TUI 画面を作ろう
tags: Go
author: zetamatta
slide: false
---
本文書は [Go5 Advent Calendar 2019](https://qiita.com/advent-calendar/2019/go5)、５日目の記事です。Go5-5！

さて、最近、コマンドプロンプトで動作する [CSVビューア](https://github.com/zetamatta/csview) とか、[ツイッタクライアント](https://github.com/zetamatta/tmt) を書いてます。

この手のツールを書く時、本格的なフレームワークライブラリもいいのですが、欧米人の書いたやつはなんか表示のズレが多く、めんどくさいことが多い（大偏見）ので自分は

* [go-tty](https://github.com/mattn/go-tty) … キーコード入力ライブラリ
* [go-colorable](https://github.com/mattn/go-colorable) … ANSIエスケープシーケンスエミュレーター
* [go-runewidth](https://github.com/mattn/go-runewidth) … 文字がコンソールで何セル使用するかを得る

という三種の神器的なライブラリでやることが多いです。この３セットで書いてたら、普通に Windows / Linux 双方対応でき、お手軽に OS への依存性を除くことができるので、自分は重宝しています。

が、それでも画面崩れるんですよね。[ツイッタクライアント](https://github.com/zetamatta/tmt)で画面を見てると、たまに１行に文字を出力しすぎて画面がスクロールしてしまう。調べてみると、一部の文字が原因。そう文字幅が想定と違うんです。

一例をあげると、たとえば「&#x2727;」(`U+2727` : `WHITE FOUR POINTED STAR`)。これ、[Unicode](https://unicode.org/Public/12.1.0/ucd/EastAsianWidth.txt) の規格では１セル分扱いのハズなんですが

**自宅のWindows10でのスクリーンショット**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/29454/7f12f575-dc47-7fe9-6e8d-c8b1a459f48c.png)
→ 1セル分しかカーソルが移動しない（%U+nnnn% は nyagos の拡張機能）

**会社のWindows7でのスクリーンショット**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/29454/e3c1cb07-f5c3-db0b-1a47-4bd90eaded0a.png)
→ 2セル分カーソルが移動している

環境によって表示した時のカーソル移動量が違うんですね。これはどうしたものかと思っていたら、先生から啓示が出ました「フォントです」「フォントですか」。つまり、Windows 7だからこの幅、10だからこの幅みたいな判断もできない。

ここに Windowsコマンドプロンプト向けのバッタもん runewidth( go-lunewhydos という名前まで考えてた）開発計画は一旦頓挫したのでした。

### ならば、端末ごとにデータベース作ればいいんじゃね？

いわゆる「キャリブレーション」というワードが脳内にポップアップしました。昔、ブラウン管のモニターの表示を調整するのに、ユーザオペレーションで画面位置を調整していましたし、今でも PS4 のゲームとかで表示調整する時に「龍がギリギリ見えるところを選択してください」みたいなことをやりますよね。ユーザ操作まではいらないけども、端末ごとに１アクションして、結果をデータベースとして残すようにすればよいのです。

ということで、Unicode での公式幅と違う幅の文字を検出し、それを参照するライブラリ [go-termgap](https://github.com/zetamatta/go-termgap) （端末のギャップ）を作成しました。

### 原理

* 「`\r` ＋目的の文字」を表示してから、カーソル位置を調べる」ということを全Unicode文字に対して行う
    * ただし、サロゲートペアな文字については、そもそも Windows のコマンドプロンプトでは表示できないので、省いてよい

わかりやすいな

### どないして、カーソル位置を取得すんねん

Goから OS の API を呼ぶというと、一般には `import "cgo"` を使うことが普通とされています。ですが、これ C言語が標準装備された UNIX ではよいのですが、Windows では要件が増えてビルドするための敷居があがってしまいます。

Windows の場合、それよりも `"syscall"` を使って、DLL をロードして、そちらから API を呼び出すのがよいでしょう。これならビルド要件はあがりません。

で、さらに最近は、そこまでせずとも、実は `"golang.org/x/sys/windows"` に既に多くの DLL 関数が定義されていて、そこに定義されていたら、それを使うだけで済んでしまいます。

それを使って実装してみたのが、こちら：

```go
func X()(int,error){
    handle, err := windows.GetStdHandle(windows.STD_ERROR_HANDLE)
    if err != nil { return 0 ,err }

    var buffer windows.ConsoleScreenBufferInfo
    windows.GetConsoleScreenBufferInfo(handle, &buffer)
    return int(buffer.CursorPosition.X),nil
}
```

さて、この文字幅計測関数を、サロゲートペアになっていない全ユニコード（※）で実際に動かしてみましょう！

![demo.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/29454/be5c5990-bf70-cc90-9cc8-43c5eee12115.gif)

思ったほど時間がかかりませんでした。１分もかかってない。最初に１回やっておくだけでいいならば十分許容範囲です。これをどこかに保存しておけばよいわけです。普通に `"encoding/json"` でも使っておきましょうか。

これをどこにおくか…ですが、Go言語にはユーザ向けのディレクトリを得る関数が３つ用意されています。

```go
func os.UserCacheDir() (string, error)
func os.UserConfigDir() (string, error)
func os.UserHomeDir() (string, error)
```

Windows では、

* `os.UserCacheDir()` は `%USERPROFILE%\AppData\Local`
* `os.UserConfigDir()` は `%USERPROFILE%\AppData\Roaming`

となっていることが多いようです。Roaming フォルダーは、例えば PC を移行した時にユーザに紐づく情報としてもってゆくデータを入れる場所らしいです。こんな端末に紐付いたデータは置くべきではないので、Local の方にフォルダーを掘って、そこに保存することにしましょう
（ → `%USERPROFILE%\AppData\Local\nyaos_org\termgap.json` とした）

※（2020/1/3追記）Windowsのフォルダーの使いみちについて詳しい記事がありました。→ [Windowsのディレクトリ構成ガイドライン - torutkのブログ](https://torutk.hatenablog.jp/entry/20110604/p1)

### ライブラリ利用編

では、作ったライブラリを利用してみましょう！

```go
    db, err := termgap.New()
    if err != nil {
        return err
    }
    w, err := db.RuneWidth('\u2727')
    if err != nil {
        return err
    }
    fmt.Printf("[\u2727]'s width=%d.\n", w)
```

**めんどくさ！** いちいち、インスタンス作らなきゃいけないのかよ！しかも、文字幅データベース（JSON ファイル）が作成されてなかったら、`New()` でエラー！？**つかえん！**

というわけで、ラッパーライブラリを作りました。これならどうでしょう。

```go
import (
    "fmt"
    "github.com/zetamatta/go-termgap/hybrid"
)

func main() {
    fmt.Printf("[A]'s width=%d\n", hybrid.RuneWidth('A'))
    fmt.Printf("[\u2727]'s width=%d\n", hybrid.RuneWidth('\u2727'))
}
```

はい、インラインで使えますね。これなら許容範囲です。

でも、文字幅データベースが作られてない環境だとどうなるんだ？Panic ?

ノンノン、Panic など愚の骨頂。文字幅データベースがないなら、かわりにどっかからデフォルト値をもってくればよいのです。たとえば[人様のライブラリ](https://github.com/mattn/go-runewidth)を使って…（悪い顔）

※ だから、パッケージ名が hybrid なんです、はい

なお、Linux の場合ですが、キャリブレーションの方はともかくとして、データベースを利用する側は JSON ファイルが見つからず、普通に go-runewidth にぜんぶ丸投げするだけなので、特にビルド上の問題などはありません。

以上、人様のライブラリにおんぶにだっこしたい同志各位のご参考になれば幸いです。

