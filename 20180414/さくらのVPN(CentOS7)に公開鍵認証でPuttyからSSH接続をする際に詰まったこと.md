---
ID: 143bf928fb7103ca0c96
Title: さくらのVPN(CentOS7)に公開鍵認証でPuttyからSSH接続をする際に詰まったこと
Tags: VPN,さくら,centos7
Author: ""
Private: false
---

#前置き
さくらのサーバーに公開鍵認証する手順は大体以下のようになる。
1. 公開鍵と秘密鍵のペアを作成する(Puttygen.exe)
2. 秘密鍵をローカルに、公開鍵をサーバ側に保存する
3. /etc/ssh/sshd_configを書き換える
(<font color = red>**ポートの変更**</font>、パスワード認証の停止、rootでのログイン禁止、秘密鍵認証の有効化）
4. 秘密鍵のディレクトリパスをPutty側で指定して、サーバにアクセスする

この手順についてはQiitaのこちらの記事が詳しい
https://qiita.com/sango/items/816136188387221f05b3

ただし、このままではSSH接続ができなかった。ポートを22番ポートに戻し、22番ポートにSSH接続を行うと接続ができる。

#結論

結論を言うと、私のサーバではFirewallが動いていて、22番ポート以外への接続を全て弾いていた。

https://webkaru.net/linux/centos7-firewalld-ssh-port/

この記事を参考にファイアウォール側で設定を変更し、22番ポートを閉じて、SSHで接続したいポート(仮に50000番)を開いた。

#問題の切り分け
ファイアウォールが起動しているかは、

```
$systemctl status firewalld.service
```

で確認ができる。active(runnning)と緑色で表示されれば起動している。

![キャプチャ.PNG](https://qiita-image-store.s3.amazonaws.com/0/124948/cc71cf88-20cb-048f-e6c4-df69dc2ef1e4.png)
こんな感じ。

Firewallが起動していないのに設定したポートへのSSH接続ができない場合は、他の原因を疑ったほうがいいかもしれない


次は、起動しているファイアウォールが本当に原因かどうか。

友人によると、こういうときはFirewallを一回落としてみて、今回設定したいポート番号（例えば50000）に接続して、接続できるかを確認するのがいいらしい。たしかに。

というわけで、ファイアウォールを落として、SSH接続をしてみる。
コマンドは

```
$systemctl stop firewalld.service
```

これでも、SSH接続ができない場合は、ファイアウォール以外にも原因がある可能性がある。他の原因も疑ってみよう。

#VPN側で弾かれている可能性
半年後、2台目のサーバ構築の際、別の点で引っかかったのでメモ

VPNで設定したポート許可のポリシーで弾かれている可能性がある。確認して、全許可してfirewalldを適切に設定したり、
コンソール画面で適切なポリシーを設定したりして対応しよう。

#鍵に関して

`openssh -t rsa -f authorized_key`を使うのおすすめ。
permissionは`.ssh/`が600, `authorized_key`は700。
puttygenで作ってuploadしてもいいんだけど、ppk形式からrsa形式に変換する手間が発生する（慣れてないだけ）

---
今回の記事は以上。
