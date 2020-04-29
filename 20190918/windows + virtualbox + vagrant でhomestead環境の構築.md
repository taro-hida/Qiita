---
ID: abc572ac050a5a48d211
Title: windows + virtualbox + vagrant でhomestead環境の構築
Tags: homestead
Author: ""
Private: false
---

windows環境にHomestead環境をインストールする手順です。

# TL;DR

ほとんど[参考サイト](#参考)の手順に則っていますが、[homestead.ymlの編集](#homesteadymlの編集) の箇所だけUnix向けで、windowsでは調整が必要だったので公開します。

# 前提となる環境
各々インストールしている状態を前提とします。インストール方法については、[参考サイト](#参考)に記載があります。

- Git
- VirtualBox
- Vagrant

動作環境はwindows10 Homeです。

# Homestead環境の構築
コマンドプロンプト上で作業をします。作業はホームディレクトリを使用中。

##インストール
今回は最新リリースのv9.1.0を使用します。好きなバージョンを選んでいただければ。
ここも、指定するバージョンを除けば[参考サイト](#参考)と変わりはありません。

```cmd
> vagrant box add laravel/homestead
> git clone https://github.com/laravel/homestead.git Homestead
> cd Homestead
> git checkout v9.1.0
> init.bat
```

##Homestead.ymlの編集

Homestead.ymlを以下のように書き換えます。
デフォルトはUnixの形式になっているので、windowsのフォルダ形式で指定しなおす必要があります。

\<user_name\>の箇所は、各々のユーザで書き換えてください。
vimで開いて、`:%s/<user_name>/taro_hida/g`とか使うと便利です。

```
---
ip: "192.168.10.10"
memory: 2048
cpus: 1
provider: virtualbox

authorize: C:\Users\<user_name>\.ssh\id_rsa.pub

keys:
    - C:\Users\<user_name>\.ssh\id_rsa

folders:
    - map: C:\Users\<user_name>\Homestead\code
      to: /home/vagrant/code

sites:
    - map: 192.168.10.10
      to: /home/vagrant/code/public

databases:
    - homestead

# ports:
#     - send: 50000
#       to: 5000
#     - send: 7777
#       to: 777
#       protocol: udp

# blackfire:
#     - id: foo
#       token: bar
#       client-id: foo
#       client-token: bar

# zray:
#  If you've already freely registered Z-Ray, you can place the token here.
#     - email: foo@bar.com
#       token: foo
#  Don't forget to ensure that you have 'zray: "true"' for your site.
```

`vagrant up`発行してしばらく待機し、プロンプトが返ってきたら`vagrant ssh`でログインしましょう。
sshが利用できるなら、`ssh vagrant@192.168.10.10 -i C:\Users\<user_name>\.ssh\id_rsa`でログインできます。
もちろん、RLoginなどのターミナルソフトからsshログインすることも可能です。

```
> vagrant up
..(略)..
> vagrant ssh
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/e83038bc-ba38-e8f0-5c8b-7e038c1c2c5d.png)

↑ ログイン後の画面。こういうAA出力すき

# 参考


- [Laravel開発のはじめかた Windows編](https://www.hypertextcandy.com/start-laravel-project-on-windows)
