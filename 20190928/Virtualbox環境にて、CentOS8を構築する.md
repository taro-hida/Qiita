---
ID: fa05da9c7541bd825981
Title: Virtualbox環境にて、CentOS8を構築する
Tags: VirtualBox,centos8
Author: ""
Private: false
---

本記事はCentOS8をVirtualbox環境にインストールしたい人向けの記事です。

# TL;DR

基本的には画面の指示に従っていけば問題なくインストールできますが、いくつか罠があったのでそこだけ注意。

# 環境
Window 10 Home
Virtualbox 5.2.8r
# 手順　

## dvd.isoをローカルPCにダウンロードする
[CentOS公式ダウンロードページ](https://www.centos.org/download/)から。
6.6GBあるので、ゆっくり待ちましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/87b265b2-d52c-5466-e7c2-212359995446.png)


## Virtualbox上でインストール

DVD isoをVirtualboxのドライブにセットし、仮想環境を立ち上げてインストールします。
基本的に、画面の指示に従っていけば問題なくインストールできます。

この方が、スクショ付きで解説されているのでご参考まで。本記事ではスクショは割愛します。
[CentOS8 OSインストール/cockpit/リポジトリ追加/Ansible/snappy](https://qiita.com/tukiyo3/items/d87c8752f2b59f73d062)

以下、Virtualboxでインストールする際の注意点です。
### 注意点1
Virtualboxでの仮想サーバ作成時、
メモリはデフォルト1GB(1024MB), ディスクは8GBですが、
CentOS8のメモリ推奨は1.5GB、ディスク必要容量は8GB強あります。

メモリ1GBで怒られるかどうかはわかりませんが、ディスク8GBだと自動パーティションに容量が足りないと怒られます（手動でパーティション調整すればいけるような気がします）

初回の仮想環境作成時に、ディスクは16GB程度確保しておくようにしましょう。

### 注意点2
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/f7f23dcc-9491-5b9a-3f07-c556fd64d332.png)

「Server with GUI」を選択してCentOS8をインストールすると、インストールが正常に完了したのち、OSが立ち上がってこなくなります。
「Server」を選択してインストールを行いましょう。

ありがとうございました -> [CentOS 8 リリース待ち→リリースされた2019-09-24(^^)](https://tech.godpress.net/?p=1071)

## Hello, CentOS8 !
これでコンソール画面からの接続ができるようになったはずです。

## ホスト端末からSSH接続を行う
RLogin等のターミナルソフトを使って、SSH接続をしてみましょう。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/d3722aa0-88a1-8da3-7c59-9698686e8a5e.png)
設定> ネットワーク > ポートフォワーディングから、いつものようにポートフォワードの設定を入れます。

Ubuntuの場合はこれでよかったですが、CentOSの場合はこれに加え、ネットワークインターフェースの有効化が必要みたいです。

`/etc/sysconfig/network-scripts/`以下の、`ifcfg-enp0s3`等ファイルを編集します。

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=d3780950-2513-4e68-8c97-ffd4567ce20e
DEVICE=enp0s3
ONBOOT=yes
```
ONBOOTをyesに変更して保存し、サーバをリブート。
ありがとうございました -> [VirtualBox に入れた CentOS のネットワーク設定](https://www.shookuro.com/entry/2018/02/10/172724)

後は、sshコマンド発行すれば入れます。
今回のポートフォワードの設定であれば、発行するコマンドは以下のようになります。

```
ssh root@127.0.0.2 -p 2222
```

## 日本語入力を有効にする

日本語の言語パックをインストールしましょう。日本語の文字化けを防ぐことができます。
dnfのエラーコードを解消することもできます（[詳細リンク](https://qiita.com/takizawafw6o2o/items/d6a15b5ae05d85ae1f41)）。

```
# sudo yum -y install langpacks-ja
```

# 参考

- [VirtualBox に入れた CentOS のネットワーク設定](https://www.shookuro.com/entry/2018/02/10/172724)
- [CentOS 8 リリース待ち→リリースされた2019-09-24(^^)](https://tech.godpress.net/?p=1071)
