---
ID: d65079adcfc44511e3bf
Title: shellscriptによるsshログインの自動化
Tags: Bash,Expect,shell,rssh,assh
Author: ""
Private: false
---

少し前の記事で、ssh \<tab>とtabをたたくと、.ssh/configに記載したホスト名で補完してくれるscriptができ、ちょっと丁寧めに紹介しました。
-> https://qiita.com/hidaro/items/384cc8dd156364cb035f

これはそれの続きとなります。次はsshログインを自動化しましょう。

```~/assh.sh
#!/bin/bash

passwd=`grep '  #'$1'pass' ~/.ssh/config | awk '{print $2}'`
cr="\n"

expect -c "
  spawn ssh $1
        expect \"password:\"
        send $passwd$cr
  interact
"
```

このスクリプトに関してはほぼ以下の記事を見れば内容がよくわかると思います。
http://www.uetyi.com/server-const/entry-158.html
expectに関して非常にわかりやすかった記事です。

加えて、"assh"と叩くだけで実行できるように、.bashrcにはエイリアスを追記しておきます（assh = Auto SSHの略）

```~/bashrc.
alias assh=". ~/assh.sh"
```
ちなみにassh.shのパスは好きなとこに置き換えて大丈夫です。

sshに自動でログインするためには、ホストに対応したpasswordが必要です。
このpasswordは、~/.ssh/configに記載しています。

```~/.ssh/config.
Host sshhost
        HostName hogehost
        User hogeuser
        Port 22
        #sshhostpass hogepassword
```
※独自に編み出した方式なので、もっといいのあったら教えてください

ちなみに`assh.sh`においては、

```assh.sh
passwd=`grep '  #'$1'pass' ~/.ssh/config | awk '{print $2}'`
```
この一文でpasswordをgrepして抽出しています。

とはいえ、いちいちこんなものを`~/.ssh/config`に記入していくのは面倒すぎますね。

登録を楽にするためのスクリプトも用意しました。

```rssh.sh
#!/bin/bash

echo -n 'Host -> '
read Host
echo -n 'username -> '
read username
echo -n 'hostname(FQDN or IP) -> '
read hostname
echo -n 'port -> [22] '
read port
if [ -z "$port" ] ; then
  port='22'
fi
echo -n 'password ( or passfrase ) -> '
read password
if [ -z "$password"] ; then
  password='defaultpassword'
fi
echo -n 'key path -> [] '
read keypath
if [ -z "$keypath"] ; then
  keypath='none'
fi

echo >> .ssh/config
echo 'Host '$Host >> .ssh/config
echo '  HostName '$hostname >> .ssh/config
echo '  User '$username >> .ssh/config
echo '  Port '$port >> .ssh/config
if [ -n "$keypath" ] ; then
  echo '        IdentityFile '$keypath >> .ssh/config
fi
echo '  #'$Host'pass '$password >> .ssh/config

cr="\n"

expect -c "
  spawn ssh $Host
        expect \"password:\"
        send $password$cr
  interact
"
```
対話式で~/.ssh/configの内容を作成できるscriptです（rssh = Register SSHの略）。

ちなみに情報が正しければそのままログインできます。
consoleへの出力はこんな感じ

```console.
user@sshcliant:~$ rssh
Host -> test
username -> hogeuser
hostname(FQDN or IP) -> hogehost
port -> [22]
password ( or passfrase ) ->
key path -> []
spawn ssh hogehost
hogeuser@hogehost's password:
Last login: Sun Aug  5 15:46:18 2018 from sshclient
```
`.ssh/config`への追記内容はこんな感じ。

```.ssh/config.
Host test
        HostName hogehost
        User hogeuser
        Port 22
        IdentityFile none
        #testpass defaultpassword
```

これで、rsshで登録し、その後はassh \<hostname>と打つと自動ログインできるようになりました。
しかも assh \<tab>で自動補完が効きます。

最後まで読んでくださいましてありがとうございました。
お役に立てていれば幸いです。

次は鍵認証にも対応したいですね。とはいえexpectで分岐させるだけなので簡単にできるはず...

#鍵認証も実装しました。

```~/assh.sh
expect -c "
  spawn ssh $1
  expect \"password:\" {
    send $passwd$cr
  } \"passphrase\" {
    send $passwd$cr
  }
  interact
```
