---
ID: 468681396888ede094f0
Title: bashのプロンプト表示を管理する便利ツールを作ったので紹介する
Tags: Bash,dotfiles
Author: ""
Private: false
---

この記事は [dotfiles Advent Calendar 2019](https://qiita.com/advent-calendar/2019/dotfiles) 17日目の記事です。[^1]

今回は、bashのプロンプト表示を簡単に管理する便利ツール（エイリアス群）、
`promptrc`について紹介します。

# promptrcの紹介

## 導入方法
[promptrcのGithubリンク](https://github.com/taro-hida/dotfiles/blob/master/promptrc)

コードとしては以下となります。

```bash
# 必要な変数定義部分
export BLACK="$(tput setaf 0)"
export RED="$(tput setaf 1)"
export GREEN="$(tput setaf 2)"
export YELLOW="$(tput setaf 3)"
export BLUE="$(tput setaf 4)"
export MAGENTA="$(tput setaf 5)"
export CYAN="$(tput setaf 6)"
export WHITE="$(tput setaf 6)"
export RESET="$(tput sgr0)"

# プロンプト表示設定部分
export PS1='[\A ${GREEN}\u${RESET}${CYAN}@\H${RESET} \W]\n\$ '
```
sourceコマンドで読み込むと反映されます。

```bash
$ source promptrc
```
`.bashrc`にsourceコマンドを記載しておくことで、ログインするたび読み込むことが可能です。

```
# bashrcに追記
source ~/promptrc
```

`dotfiler`[^2] の方は、bashrcに読み込む旨を追記してください。
で、`promptrc`をadd, commit, pushして、Gitで管理してあげてください。


```bash
# bashrcに追記
source ~/dotfiles/promptrc
```

デフォルトだと、プロンプト表示は以下のようになります（私のお気に入りの表示です）。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/0b5df652-1f0e-4b9a-ac25-ca864b75df81.png)

## 利用方法

### 色のカスタマイズ
色を変更したい場合は、`${COLOR}`で指定します。
試しに、デフォルトを以下のように書き換えてみましょう。
書き換えたらsourceで再読み込みしてください。

```diff
- export PS1='[\A ${GREEN}\u${RESET}${CYAN}@\H${RESET} \W]\n\$ '
+ export PS1='[\A ${RED}\u${RESET}${YELLOW}@\H${RESET} \W]\n\$ '
```

なお、とりあえず一時的に変更したい場合は、以下のように環境変数を設定しなおすだけで問題ありません（動作確認など）

```
$ PS1='[\A ${RED}\u${RESET}${YELLOW}@\H${RESET} \W]\n\$ '
```

以下のように表示が変更されました。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/be8cf612-58b1-05e8-0bdb-8d376718eca2.png)

### 表示のカスタマイズ
表示内容についても同様に変更できます。パラメータについては[こちらのブログ](http://pocketstudio.jp/linux/?%A5%D7%A5%ED%A5%F3%A5%D7%A5%C8%A4%CE%B3%CE%C7%A7%A4%E4%C0%DF%C4%EA)が詳しいです。
以下引用します。

> |    |                                                                              |
|----|------------------------------------------------------------------------------|
| \a | ベル（ビープ音）をならします（ASCII のベル文字 07）                          |
| \d | "曜日 月 日"の形式の日付                                                     |
| \h | ホスト名(最初の . までの名前)                                                |
| \H | ホスト名                                                                     |
| \n | 改行                                                                         |
| \r | 復帰                                                                         |
| \s | シェル名（標準だと -bash が表示）                                            |
| \t | 時刻 HH:MM:SS 形式(24時間) H = Hour = 時、M = Minutes = 分、S = Seconds = 秒 |
| \T | 時刻 HH:MM:SS 形式(12時間)                                                   |
| \@ | 時刻 am/pm をつけたもの。Lang=Jaの場合 HH:MM (午前|午後)となる。             |
| \u | 現在のユーザー名                                                             |
| \v | bash のバージョン                                                            |
| \V | bash のバージョン・リリース番号など詳細                                      |
| \w | 現在のディレクトリ（フルパス）                                               |
| \W | 現在のディレクトリ名                                                         |
| \! | コマンドのヒストリー番号                                                     |
| \# | コマンドのコマンド番号（ログイン後何回実行したか）                           |
| \$ | UID が 0 であれば『 # 』、それ以外は『 $ 』                                  |
| \\ | バックスラッシュ                                                             |
| \[ | 表示されない文字列（エスケープシーケンス／端末制御シーケンス）を埋め込む     |
| \] | 表示されない文字列の終わり                                                   |


では出力内容についてカスタマイズしてみましょう。
日付出力 + 時刻について詳細に表示するようカスタマイズを行います。

```diff
- export PS1='[\A ${RED}\u${RESET}${YELLOW}@\H${RESET} \W]\n\$ '
+ export PS1='[\d \t ${RED}\u${RESET}${YELLOW}@\H${RESET} \W]\n\$ '
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/675841db-60ad-89e3-177f-ecc59cb93dad.png)

dateコマンドの結果をそのまま出力することも可能です。
自作コマンド等の出力を利用することももちろん可能です。

```diff
- export PS1='[\d \t ${RED}\u${RESET}${YELLOW}@\H${RESET} \W]\n\$ '
+ export PS1='[`date`${RED}\u${RESET}${YELLOW}@\H${RESET} \W]\n\$ '
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/3799f67b-fa2d-89ef-93af-ba598680f218.png)


# 何がよいのか
やっていることとしては、色のパラメータを変数にまとめただけです。
`\[\033[31m\]`等のパラメータを直書きしている記事が多く、最初は私もそれでやってましたが
変更の際いちいちググる必要があり面倒でした。

変数で指定するようなものが見当たらなかったため自作しました。

# まとめ
- プロンプトのカスタマイズは気持ちいい
- `\[\033[31m\]`とかで指定すると面倒なので、変数にまとめておこう。
- 各々が調べて車輪の再発明するのはアレなので、この機会に公開しよう！

# おわりに
[dotfiles Advent Calendar 2019](https://qiita.com/advent-calendar/2019/dotfiles) に参加させていただき、ありがとうございました。

キーボードやマウス、端末にこだわる方は多くても、プロンプトにこだわる方は少なかったりするのではないでしょうか？

一般に道具に拘ると愛着が増して、使うのが楽しくなってくるものと思います。
プロンプトもカスタマイズすると愛着が沸いてかわいく思えてきます。

ターミナルでの作業が楽しくなるので、未カスタマイズの方はぜひカスタマイズしてみてください。
紹介したツールを導入していただければ、カスタマイズおよび管理のハードルが下がると思います。

次回は18日目、@ahuglajbclajepさんです。

# 補足

- [EZPROMPT](http://ezprompt.net/)という便利なツールがありました。([Qiitaの紹介記事はこちら](https://qiita.com/kkenya/items/f7cb4b8787e7ac27a46d#ezprompt))

- こんなネタプロンプトの作成も可能です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/9691a799-4e05-6cdc-cc52-f92540db41dc.png)

- `PS1=''`とか発行するとかなり異様な操作感が味わえます。元の世界に帰りたい場合は`exit`を発行してください。

[^1]: 一度言ってみたかったんですよねこれ
[^2]: 正式な呼び方を存じておりません。界隈標準の呼称とかあるんでしょうか
