---
ID: 4980dd1cc6ce07c051cf
Title: '"LF"のF(Feed)でなぜ改行を表すのか'
Tags: ascii
Author: ""
Private: false
---

# はじめに

ASCIIコードにおいて"0x0A"は"LF", "0x0D"は"CR"を表します。
"CR"は"Carrige Return"で、Carrigeをもとの位置に戻すという意味です。これはそのままの意味で理解しやすいです。

"LF"については、"Line Feed"ですが、"Feed"というのは家畜に餌を与えるニュアンスの強い言葉です。
何かを与えるという意味ではFeedを適用できるように思いますが、なんだかすごくもやもやしました。

# 割と納得したニュアンス

このサイトで私と同じ疑問を持った人の質問がされていました。
https://ell.stackexchange.com/questions/148702/the-origin-of-feed-in-line-feed?newreg=0915c3c567624e4ba5290d8070db6a45

Type Writerに紙を与えるというニュアンスです。
Type Writerの作動原理ですが、紙の乗ったCarrigeを動かして印字位置を移動させるというものとなります。
Type Writerに紙をfeedして、lineについてもfeedして、というニュアンスです。

まあ結局のところ母国話者じゃないのでわからないんですが、すっきりしたので良しとします。
あと、この記事にたどり着くまでに2時間くらいかかったので、共有の意味も込めてこの記事を書きます。
