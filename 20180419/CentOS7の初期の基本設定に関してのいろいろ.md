---
ID: 518e2b6933887d4d00e1
Title: CentOS7の初期の基本設定に関してのいろいろ
Tags: 自分用メモ,centos7
Author: ""
Private: false
---

※読み方の説明
[username]など、[ ]で囲んだ箇所は各々の環境に応じて読み替えてください。

#ユーザを作成し、グループを作成し、グループにユーザを割り当て、グループの権限を変更する
- ユーザの作成・追加

`#useradd [username]`

※同時にグループに<b>追加</b>する場合 ※変更の場合また異なる

`#useradd -G [groupname]`

- ユーザへのPasswordの設定
 
 `#passwd [username]`

 - グループの作成

 `#groupadd [groupname]`

 - グループに権限を与える

 `#visudo`
手順：https://qiita.com/Esfahan/items/a159753d156d23baf180 を参照（sudo時passwordなしの設定）

#visudoの文法について

```/etc/sudoers
user ALL(ALL) ALL
%wheel ALL(ALL) ALL
%nopass ALL=NOPASSWD: ALL

1列目：
- hoge -> user
- %hoge -> group
2列目：
- よくわからん
3列目：
- 許可するコマンドを指定する

```
##hostnameを変更
RHEL系は`hostnamectl`を用いるのが正攻法
（らしい)
`hostnamectl set-hostname [hoge]`
： /etc/hostnameを更新するコマンド。
（/etc/hostnameを直接弄っても構わない）

[RHEL公式の該当ドキュメント](https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec_Configuring_Host_Names_Using_hostnamectl.html)

#SSH以外のポートを全部閉じる

```
# firewall-cmd --get-default-zone
  public
# firewall-cmd --list-service --zone=public
  ssh dhcpv6-client
# firewall-cmd --remove-service=dhcpv6-client --pernament
  success
# firewall-cmd --list-service
  ssh
```
firewalldのコマンド`firewalld-cmd`に関しては[ここ](https://www.server-world.info/query?os=CentOS_7&p=firewalld)を参照

上は詳しすぎるので、[こっち](https://qiita.com/miyazawa214/items/88bb5e3a8c8b2c7840a1)もいい
※これで22ポートもfirewalldで防げてるのかわからない

#SSL証明書の設定（Let's Encrypt）

Let's Encrypt公式の他、役に立ったブログ
https://qiita.com/ariaki/items/174aacfc224d2af5f2ee

使用したコマンド

※lets encryptは80番ポートを使用するので、apache等は停止させること！

```console.sh
sudo yum install certbot
sudo certbot certonly -w /var/www/html/ -d t-tona.com
sudo yum install mod_ssl
sudo vim /etc/httpd/conf.d/ssl.conf

#記述内容#############
[mutsu@t-tona conf.d]$ sudo diff -u ssl.conf.org ssl.conf
--- ssl.conf.org        2018-09-02 03:56:20.998360151 +0900
+++ ssl.conf    2018-09-02 03:32:40.903501253 +0900
@@ -97,14 +97,14 @@
 # the certificate is encrypted, then you will be prompted for a
 # pass phrase.  Note that a kill -HUP will prompt again.  A new
 # certificate can be generated using the genkey(1) command.
-SSLCertificateFile /etc/pki/tls/certs/localhost.crt
+#SSLCertificateFile /etc/pki/tls/certs/localhost.crt
 
 #   Server Private Key:
 #   If the key is not combined with the certificate, use this
 #   directive to point at the key file.  Keep in mind that if
 #   you've both a RSA and a DSA private key you can configure
 #   both in parallel (to also allow the use of DSA ciphers, etc.)
-SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
+#SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
 
 #   Server Certificate Chain:
 #   Point SSLCertificateChainFile at a file containing the
@@ -212,5 +212,11 @@
 #   compact non-error SSL logfile on a virtual host basis.
 CustomLog logs/ssl_request_log \
           "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
+
+ServerName t-tona.com
+DocumentRoot /var/www/html
+SSLCertificateFile /etc/letsencrypt/live/t-tona.com/cert.pem
+SSLCertificateKeyFile /etc/letsencrypt/live/t-tona.com/privkey.pem
+SSLCertificateChainFile /etc/letsencrypt/live/t-tona.com/chain.pem
 </VirtualHost>                                  
####################

sudo firewall-cmd --add-port=443/tcp --zone=public --permanent
sudo systemctl restart firewalld
sudo systemctl restart httpd
```

これでブラウザにアクセスすればおｋ。


#python3のインストール
https://qiita.com/estaro/items/5a9e4eb8f0902b9977e3

#python-pipのインストール
pipを別途インストールします
http://rriifftt.hatenablog.com/entry/2015/10/28/142043


#php7のインストール
https://webbibouroku.com/Blog/Article/centos-php7

###・加えて、Qiitaに載ってない外部リンクを以下にまとめる

##SSH接続関連の設定
[Firewallのポート開放・閉鎖（Port変更したときの必須手順）](http://senoway.hatenablog.com/entry/2015/02/11/142139)
[rootでないユーザーへの権限の付与](https://eng-entrance.com/linux-command-visudo#visudo)
[Conohaサーバへの初期設定もろもろ](https://hombre-nuevo.com/vps/vps0009/)
[お名前.comでの独自ドメイン設定に必須のやつ(のわりにどこにもない)](http://www.pandanoir.info/entry/2017/04/20/152129)

