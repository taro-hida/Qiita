---
ID: 1cc86a1c713e835ffb41
Title: yumが何をしているのかについて理解してみよう
Tags: Yum,CentOS7.6
Author: ""
Private: false
---


何となく使っていたけど、実際どのような動きをしているのかわからない`yum`くんについて改めて理解してみようという記事。
実際にファイルを覗いてみたりしましょう。

少し長くなったので、途中で読むのに疲れた人は、[まとめ](#まとめ)だけでも読んでいってください。

# 環境

```terminal
$ cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core)
```

#前提
rpmとyumの違いが何となくわかる人向け。

「全然わかんねぇ！！」って人は以下の記事が読みやすいのでおすすめです。 
https://qiita.com/sksmnagisa/items/05a6f8a707010b8bea56


# 用語一覧
- パッケージ ( package )
関連ファイルを一つにまとめたもの
httpdパッケージなら、`/etc/httpd`以下のファイルとか、`/var/www/html`以下のファイルとか。

- レポジトリ ( repository )
パッケージをホストするサーバ

- RPMファイル
パッケージの形式のうちの一つ。Red Hat系OS用のパッケージがRPMパッケージ。Ubuntuだとdebパッケージ。
`.rpm`っていう拡張子になる

- RPMデータベース
`/var/lib/rpm/`に存在する、rpm関連のデータを格納するデータベース。パッケージの情報を参照したりするのに利用する。

レポジトリとパッケージの関係を図にすると以下のような感じ
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/6454d1d1-7e81-a584-d34b-aaf13ae3babd.png)



## レポジトリ
レポジトリの情報は`/etc/yum.repos.d/`以下のファイルで指定している。

```terminal
$ ls -l /etc/yum.repos.d/ | head
合計 124
-rw-r--r--  1 root root 1664 11月 23  2018 CentOS-Base.repo
-rw-r--r--  1 root root 1309 11月 23  2018 CentOS-CR.repo
-rw-r--r--  1 root root  649 11月 23  2018 CentOS-Debuginfo.repo
-rw-r--r--  1 root root  630 11月 23  2018 CentOS-Media.repo
-rw-r--r--  1 root root 1331 11月 23  2018 CentOS-Sources.repo
-rw-r--r--  1 root root 5701 11月 23  2018 CentOS-Vault.repo
-rw-r--r--  1 root root  314 11月 23  2018 CentOS-fasttrack.repo
-rw-r--r--  1 root root  827  5月 19  2016 atomic.repo
-rw-r--r--  1 root root 2424 10月 25  2018 docker-ce.repo
```

`less /etc/yum.repos.d/epel.repo`を発行して、実際に中身を覗いてみるとこんな感じ

```ini:/etc/yum.repos.d/epel.repo
 [epel]
 name=Extra Packages for Enterprise Linux 7 - $basearch
 #baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch
 metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
 failovermethod=priority
 enabled=1
 gpgcheck=1
 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
 
 [epel-debuginfo]
 name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
 #baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch/debug
 metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
 failovermethod=priority
 enabled=1
 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
 gpgcheck=1
 
 [epel-source]
 name=Extra Packages for Enterprise Linux 7 - $basearch - Source
 #baseurl=http://download.fedoraproject.org/pub/epel/7/SRPMS
 metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
 failovermethod=priority
 enabled=1
 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
 gpgcheck=1
```

`enabled=1`となっているレポジトリが、`yum repolist（レポジトリ一覧の表示）`を発行したときに出力される。
上記だと[epel]、 [epel-debuginfo]、[epel-source]の3つとも有効化されている

```terminal
$ yum repolist
リポジトリー ID                           リポジトリー名                                                      状態
atomic/7/x86_64                           CentOS / Red Hat Enterprise Linux 7 - atomic                           976
base/7/x86_64                             CentOS-7 - Base                                                     10,019
docker-ce-stable/x86_64                   Docker CE Stable - x86_64                                               52
epel/x86_64                               Extra Packages for Enterprise Linux 7 - x86_64                      13,367
epel-debuginfo/x86_64                     Extra Packages for Enterprise Linux 7 - x86_64 - Debug               2,931
epel-source/x86_64                        Extra Packages for Enterprise Linux 7 - x86_64 - Source                  0
extras/7/x86_64                           CentOS-7 - Extras                                                      435
ius/x86_64                                IUS for Enterprise Linux 7 - x86_64                                    871
```


## RPMファイル
yumでインストールされたrpmファイルは、`/var/cache/yum/x86_64/7/`以下に、レポジトリごとにフォルダを分けて保存される。

```terminal
$ ls -l /var/cache/yum/x86_64/7/ | head
合計 480
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.0.1406-base
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.0.1406-centosplus
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.0.1406-extras
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.0.1406-fasttrack
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.0.1406-updates
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.1.1503-base
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.1.1503-centosplus
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.1.1503-extras
drwxr-xr-x  4 root root 4096  1月 28  2019 C7.1.1503-fasttrack
```

`base`レポジトリについてみるとこんな感じ。rpmファイルは`packages`ディレクトリの下にある。

```terminal
$ ls -l /var/cache/yum/x86_64/7/base/
合計 13672
-rw-r--r--  1 root root 6289348 11月 26  2018 6614b3605d961a4aaec45d74ac4e5e713e517debb3ee454a1c91097955780697-primary.sqlite.bz2
-rw-r--r--  1 root root 7461122 11月 26  2018 a0ec5a4708a1026db100d4799c404c9ed48a9371a4bab234a1355f86628a244a-filelists.sqlite.bz2
-rw-r--r--  1 root root  169676 11月 26  2018 bc140c8149fc43a5248fccff0daeef38182e49f6fe75d9b46db1206dc25a6c1c-c7-x86_64-comps.xml.gz
-rw-r--r--  1 root root       0  8月 26 13:06 cachecookie
drwxr-xr-x. 2 root root    4096  8月  2 09:12 gen
-rw-r--r--  1 root root     606  8月 25 22:08 mirrorlist.txt
drwxr-xr-x. 2 root root   40960  7月 10 11:46 packages
-rw-r--r--  1 root root    3736 11月 26  2018 repomd.xml
```

`yum clean`コマンドで清掃されるのはここに保存されたキャッシュファイル。
`packages`以下のrpmパッケージも削除される。

```terminal
$ sudo yum clean all
読み込んだプラグイン:fastestmirror, langpacks
リポジトリーを清掃しています: atomic base docker-ce-stable epel epel-debuginfo epel-source extras ius mysql-connectors-community mysql-tools-community mysql80-community nux-dextop remi-safe unixcommunity-vim updates
Cleaning up list of fastest mirrors
Other repos take up 41 M of disk space (use --verbose for details)
$ ls -l /var/cache/yum/x86_64/7/base/
合計 48
drwxr-xr-x. 2 root root  4096  8月 26 14:40 gen
drwxr-xr-x. 2 root root 40960  7月 10 11:46 packages
```

###rpmパッケージを`rpm -i`でインストールすると何が起こる？
- install scriptが実行される
- `/etc/`以下や`/usr/sbin/`以下等、パッケージが含むファイルが配置される
- rpmデータベースに情報が追加される

install scriptについては、以下の方法で実際に中身を見ることが可能。

```terminal
$ rpm -q --scripts httpd
preinstall scriptlet (using /bin/sh):
# Add the "apache" group and user
/usr/sbin/groupadd -g 48 -r apache 2> /dev/null || :
/usr/sbin/useradd -c "Apache" -u 48 -g apache \
        -s /sbin/nologin -r -d /usr/share/httpd apache 2> /dev/null || :
postinstall scriptlet (using /bin/sh):

if [ $1 -eq 1 ] ; then 
        # Initial installation 
        systemctl preset httpd.service htcacheclean.service >/dev/null 2>&1 || : 
fi
preuninstall scriptlet (using /bin/sh):

if [ $1 -eq 0 ] ; then 
        # Package removal, not upgrade 
        systemctl --no-reload disable httpd.service htcacheclean.service > /dev/null 2>&1 || : 
        systemctl stop httpd.service htcacheclean.service > /dev/null 2>&1 || : 
fi
postuninstall scriptlet (using /bin/sh):

systemctl daemon-reload >/dev/null 2>&1 || : 


# Trigger for conversion from SysV, per guidelines at:
# https://fedoraproject.org/wiki/Packaging:ScriptletSnippets#Systemd
posttrans scriptlet (using /bin/sh):
test -f /etc/sysconfig/httpd-disable-posttrans || \
  /bin/systemctl try-restart httpd.service htcacheclean.service >/dev/null 2>&1 || :
```
`apache`ユーザ、グループ等作って、そのあとに何かごにょごにょしてるのがわかる。なるほど。

`rpm -i`で展開された各ファイルは以下の方法で確認が可能

```terminal
$ rpm -ql httpd | head
/etc/httpd
/etc/httpd/conf
/etc/httpd/conf.d
/etc/httpd/conf.d/README
/etc/httpd/conf.d/autoindex.conf
/etc/httpd/conf.d/userdir.conf
/etc/httpd/conf.d/welcome.conf
/etc/httpd/conf.modules.d
/etc/httpd/conf.modules.d/00-base.conf
/etc/httpd/conf.modules.d/00-dav.conf
```
数が多いので、一部だけ表示。なじみの深い`/etc/httpd/conf/httpd.conf`ファイルや、`/var/www/html`ディレクトリもこのタイミングで作られる。
なるほど。

rpmデータベースに保存される情報は、例えば以下のようなコマンドを打った時に表示されるもの。

```terminal
$ rpm -qi httpd
Name        : httpd
Version     : 2.4.6
Release     : 89.el7.centos.1
Architecture: x86_64
Install Date: 2019年08月26日 14時23分17秒
Group       : System Environment/Daemons
Size        : 9817317
License     : ASL 2.0
Signature   : RSA/SHA256, 2019年07月31日 12時37分23秒, Key ID 24c6a8a7f4a80eb5
Source RPM  : httpd-2.4.6-89.el7.centos.1.src.rpm
Build Date  : 2019年07月30日 02時21分18秒
Build Host  : x86-02.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://httpd.apache.org/
Summary     : Apache HTTP Server
Description :
The Apache HTTP Server is a powerful, efficient, and extensible
web server.
```
rpmデータベースについては下で詳しく説明する。


## RPMデータベース
上記の`rpm -qi`の表示などで参照されるファイルが、RPM database。
これは`/var/lib/rpm`に存在する。

```terminal
$ ls -l /var/lib/rpm
合計 205428
-rw-r--r--. 1 root root   5857280  8月  1 06:46 Basenames
-rw-r--r--. 1 root root     16384  7月 12 05:07 Conflictname
-rw-r--r--. 1 root root   3215360  8月  1 06:46 Dirnames
-rw-r--r--. 1 root root     32768  8月 25 20:59 Group
-rw-r--r--. 1 root root     28672  8月 25 20:59 Installtid
-rw-r--r--. 1 root root     57344  8月 25 20:59 Name
-rw-r--r--. 1 root root     32768  7月 22 09:09 Obsoletename
-rw-r--r--. 1 root root 196386816  8月 25 20:59 Packages
-rw-r--r--. 1 root root   2539520  8月 25 20:59 Providename
-rw-r--r--. 1 root root    413696  7月 24 07:38 Requirename
-rw-r--r--. 1 root root    126976  8月 25 20:59 Sha1header
-rw-r--r--. 1 root root     77824  8月  1 06:46 Sigmd5
-rw-r--r--. 1 root root      8192  7月  5 04:22 Triggername
-rw-r--r--  1 root root    270336  8月 26 13:27 __db.001
-rw-r--r--  1 root root     81920  8月 26 13:27 __db.002
-rw-r--r--  1 root root   1318912  8月 26 13:27 __db.003
```

レポジトリの情報などもこのrpmデータベースから引っ張ってきている。
そのため、`/var/lib/rpm`を移動させた状態で`yum repolist`などを発行するとエラーになる。

```terminal
# rpm databaseディレクトリを移動させる
$ sudo mv /var/lib/rpm/ /root/
$ yum repolist 
エラー: /var/lib/rpm にある Package データベースを開けません。
CRITICAL:yum.main:

Error: rpmdb open failed

$ rpm -qa 
エラー: /var/lib/rpm にある Package データベースを開けません。
エラー: /var/lib/rpm にある Package データベースを開けません。
```

# まとめ

図にまとめると、yumの動作は以下のように整理できる。
`yum install`を発行した際の挙動と合わせて記載する（出力の内容からも、挙動が推察できる）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/41b8129a-df06-285f-c63b-c84c177f3a4c.png)

### ①. 有効化されているレポジトリはどれかを確認する
yumコマンドの出力におけるここ

```terminal
$ sudo yum install sl
読み込んだプラグイン:fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * atomic: mirrors.hosting.in.th
 * base: ftp-srv2.kddilabs.jp
 * epel: nrt.edge.kernel.org
 * epel-debuginfo: nrt.edge.kernel.org
 * epel-source: nrt.edge.kernel.org
 * extras: ftp-srv2.kddilabs.jp
 * nux-dextop: mirror.li.nux.ro
 * remi-safe: ftp.riken.jp
 * updates: ftp-srv2.kddilabs.jp
```

### ➁. 指定されたパッケージについて、依存関係を解決する（依存関係は、各rpmパッケージファイルに記載されている）
yumコマンドの出力におけるここ

```
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ sl.x86_64 0:5.02-1.el7 を インストール
--> 依存性解決を終了しました。

依存性を解決しました
```

### ③. 対象ファイルをダウンロードする
yumコマンドの出力におけるここ

```
=============================================================================================================================================================================================================================================
 Package                                               アーキテクチャー                                          バージョン                                                    リポジトリー                                             容量
=============================================================================================================================================================================================================================================
インストール中:
 sl                                                    x86_64                                                    5.02-1.el7                                                    epel                                                     14 k

トランザクションの要約
=============================================================================================================================================================================================================================================
インストール  1 パッケージ

総ダウンロード容量: 14 k
インストール容量: 17 k
Is this ok [y/d/N]: y
Downloading packages:
sl-5.02-1.el7.x86_64.rpm
                
```

### ④. ダウンロードしたrpmファイルをインストールする
yumコマンドの出力におけるここ

```
Running transaction check
Running transaction test
Transaction test succeeded   
Running transaction
  インストール中          : sl-5.02-1.el7.x86_64                                                                                                                                                                                         1/1 
  検証中                  : sl-5.02-1.el7.x86_64                                                                                                                                                                                         1/1 

インストール:
  sl.x86_64 0:5.02-1.el7                                                                                                                                                                                                                     

完了しました!
```
[rpmパッケージをrpm -iでインストールすると何が起こる？](#rpmパッケージをrpm--iでインストールすると何が起こる)に記載した内容の繰り返しになるが、このインストールの際に、以下の処理が実行される。

- install scriptが実行される
- `/etc/`以下や`/usr/sbin/`以下等、パッケージが含むファイルが配置される
- rpmデータベースに情報が追加される

# 参考

- [初心者の頃に知っておきたかった rpm と yum の違いと使い分け - 彼女からは、おいちゃんと呼ばれています](https://blog.inouetakuya.info/entry/20111006/1317900802) 
※詳しい記事です。ただ、rpmファイルの場所については間違ってます。
- [yumについて理解が進んでいないのでメモ](https://blog.kasei-san.com/entry/2015/10/07/085002)
※参考文献が充実していてGOOD
- [rpmとかyumとかよくわからず使ってないか？](https://qiita.com/ritukiii/items/cc22efd005a06b8d4a86)
※yumに至るまでの歴史について記載されています。


# あとがき
間違ってるとこあったらコメント、マージリクエストください！マージします！マジで！！
