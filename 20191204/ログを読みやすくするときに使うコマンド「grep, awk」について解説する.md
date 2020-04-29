---
ID: 01431e8481d1749df5a0
Title: ログを読みやすくするときに使うコマンド「grep, awk」について解説する
Tags: Bash,awk,grep
Author: ""
Private: false
---

障害対応などにおいて、大量のログから必要な情報だけを確認したい時が往々にしてある。
そんなときによく使うのが`grep`と`awk`。
今回はこの2つのコマンドについて解説しようと思う。

全体として解説した記事はたくさんあるし、長くなるので、今回は
**「ログを読みやすくしたいとき」**と、シチュエーションを絞って解説する。

# grep
みなさんおなじみのコマンド、`grep`。
基本的な使い方は以下のような形。[^1]

```
$ sudo less /var/log/httpd/access_log-20190414 | grep '192.168.0.1' | less

192.168.0.1 - - [07/Apr/2019:16:49:11 +0900] "GET /webdav/ HTTP/1.1" 404 503 "-" "Mozilla/5.0"
192.168.0.1 - - [07/Apr/2019:16:49:11 +0900] "GET /help.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.29
192.168.0.1 - - [07/Apr/2019:16:49:11 +0900] "GET /java.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.29
```
[^1]: 監視アクセスと攻撃でしかアクセスのない、僕の少し悲しいWEBサーバのログ。
上記のアクセスは攻撃によるもの。
### -vオプション
便利なのは、`-v`オプション。これを用いると、該当する行を消せる。
`/var/log/messages`見るときとかによく使う。

```
$ sudo less /var/log/messages

Nov 21 16:39:01 <user-name> systemd: Started Session 14891 of user <user-name>.
Nov 21 16:40:01 <user-name> systemd: Created slice User Slice of <user-name>.
Nov 21 16:40:01 <user-name> systemd: Started Session 14896 of user <user-name>.
Nov 21 16:40:01 <user-name> systemd: Started Session 14895 of user <user-name>.
Nov 21 16:40:01 <user-name> systemd: Removed slice User Slice of <user-name>.
```

普通に閲覧していると、ログイン履歴のログだらけで非常に見にくい。

```
$ sudo less /var/log/messages | grep -v 'Session' | grep -v 'slice' | less

Nov 21 04:48:16 <user-name> yum[14644]: 更新: golang.x86_64 1.13.3-1.el7
Nov 21 04:48:22 <user-name> yum[14644]: 更新: golang-src.noarch 1.13.3-1.el7
Nov 21 04:48:33 <user-name> yum[14644]: 更新: golang-bin.x86_64 1.13.3-1.el7
Nov 21 04:48:33 <user-name> yum[14644]: 更新: golang.x86_64 1.13.3-1.el7
Nov 21 04:48:34 <user-name> yum[14644]: 更新: php72-php-pear.noarch 1:1.10.10-1.el7.remi
```
といった感じにすると、ログイン履歴のログについて消した状態で閲覧できる。
`-E`オプションをつけることで拡張正規表現が使えるようになるので、同様の処理が以下のような、

```
sudo less /var/log/messages | grep -Ev 'Session|slice' | less
```
というコマンド発行の仕方でも可能。
パイプでつなげて繰り返し目的外のログを消していけば、必要な情報を探しやすくなる。

### 正規表現

正規表現についてもよく使う。正規表現についての説明については[他のサイト](https://bi.biopapyrus.jp/os/linux/grep.html)に譲るとして、よく使う正規表現を紹介する。

```
# 11/21 16:31~35までのログを抜粋する
$ sudo less /var/log/messages | grep -E 'Nov 21 16:3[1-5]'
```

# awk

あまり使われていない気がするのが`awk`です。
`awk`くんは`grep`に比べると若干とっつきにくいので敬遠されがちですが、使えるようになると非常に楽ができるのでぜひ使いましょう。

### 基本

基本のコマンドの形式はこんな形。

```
sudo less /var/log/httpd/access_log-20190414 | awk '{print $1}' | less
```

さっきの不正アクセスのログを10行ほど抜いてみる。

```
$ sudo less /var/log/httpd/access_log-20190414 | grep '192.168.0.1' | tail -n 10
192.168.0.1 - - [07/Apr/2019:16:51:36 +0900] "GET /program/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:36 +0900] "GET /shopdb/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:36 +0900] "GET /phppma/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:37 +0900] "GET /phpmy/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:37 +0900] "GET /mysql/admin/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:37 +0900] "GET /mysql/dbadmin/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:38 +0900] "GET /mysql/sqlmanager/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:38 +0900] "GET /mysql/mysqlmanager/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:38 +0900] "GET /wp-content/plugins/portable-phpmyadmin/wp-pma-mod/index.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:51:39 +0900] "GET /manager/html HTTP/1.1" 404 503 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:31.0) Gecko/20100101 Firefox/31.0"
```

`$1`と指定するとIPアドレスが抜ける。

```
$ sudo less /var/log/httpd/access_log-20190414 | awk '{print $1}' | less
192.168.0.1
192.168.0.1
192.168.0.1
192.168.0.1
192.168.0.1
192.168.0.1
192.168.0.1
192.168.0.1
192.168.0.1
192.168.0.1
```

`$7`を指定するとリクエストしたPathが抜ける。

```
$ sudo less /var/log/httpd/access_log-20190414 | grep '192.168.0.1' | tail -n 10 | awk '{print $7}' | less
/program/index.php
/shopdb/index.php
/phppma/index.php
/phpmy/index.php
/mysql/admin/index.php
/mysql/dbadmin/index.php
/mysql/sqlmanager/index.php
/mysql/mysqlmanager/index.php
/wp-content/plugins/portable-phpmyadmin/wp-pma-mod/index.php
/manager/html
```

`$9`を指定するとステータスコードが抜ける。

```
$ sudo less /var/log/httpd/access_log-20190414 | grep '192.168.0.1' | tail -n 10 | awk '{print $9}' | less
404
404
404
404
404
404
404
404
404
404
```

便利だね。

### if 処理
この辺から本題

```
$ sudo less /var/log/httpd/access_log-20190414 | grep '192.168.0.1' | awk '$9 == 404 {print $0}' | less
192.168.0.1 - - [07/Apr/2019:16:49:11 +0900] "GET /java.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:49:12 +0900] "GET /_query.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:49:12 +0900] "GET /test.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:49:12 +0900] "GET /db_cts.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:49:12 +0900] "GET /db_pma.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:49:13 +0900] "GET /logon.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:49:13 +0900] "GET /help-e.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
192.168.0.1 - - [07/Apr/2019:16:49:13 +0900] "GET /license.php HTTP/1.1" 404 503 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
```
こんな感じに指定すると、`$9`（=ステータスコード）が`404`の時のアクセスログがきれいに抜ける。

```
$ sudo less /var/log/httpd/access_log-20190414 | grep '192.168.0.1' | awk '$9 == 404 {print $0}' | wc -l
448
```
`wc -l`をつければ`404`のアクセス数が出てくるし、

```
$ sudo less /var/log/httpd/access_log-20190414 | grep '192.168.0.1' | awk '$9 == 404 {print $7}' | less
/robots.txt
/App7a995f96.php
/webdav/
/help.php
/java.php
/_query.php
/test.php
/db_cts.php
/db_pma.php
/logon.php
/help-e.php
/license.php
/log.php
/hell.php
/pmd_online.php
/x.php
/shell.php
/htdocs.php
/desktop.ini.php
/z.php
```
こんな感じにすれば、404が発生したリクエストの要求したPathが見られる。


# 最後に

ログに出てきた `192.168.0.1`のIPは、元は本当の攻撃元IPで書いていました。
一回そのままの生ログで記事を書いてから、最後に`sed`で記事にします。

**ドン！**

```
$ cat nama.txt | sed s/xxx.xxx.xxx.xxx/192.168.0.1/ > kiji.txt
```

もいっちょ

**ドン！**

```
$ cat kiji.txt | sed s/user-name/\<user-name\>/ > real-kiji.txt
```

完成。[^2]
[^2]: 一応ユーザ名も隠しておくことにした。user-name@でSSHログインできるわけではない。
