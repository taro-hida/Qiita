---
ID: f402e0439acebb013a75
Title: CentOS7, php7.2で、mbstringを有効にする
Tags: centos7,PHP7.2,mbstring
Author: ""
Private: false
---

ネットで調べて出てくる、以下のような手順ではうまく動かなかった。

```
# yum install --enablerepo=remi,remi-php72 php-mbstring
```

入っているパッケージについて確認すると、以下の通り。

```
$ sudo yum list installed | grep php
php.x86_64                       7.2.23-1.el7.remi                   @remi-php72
php-cli.x86_64                   7.2.23-1.el7.remi                   @remi-php72
php-common.x86_64                7.2.23-1.el7.remi                   @remi-php72
php-gd.x86_64                    7.2.23-1.el7.remi                   @remi-php72
php-json.x86_64                  7.2.23-1.el7.remi                   @remi-php72
php-mbstring.x86_64              7.2.23-1.el7.remi                   @remi-php72
php-mysqlnd.x86_64               7.2.23-1.el7.remi                   @remi-php72
php-pdo.x86_64                   7.2.23-1.el7.remi                   @remi-php72
php-pecl-mcrypt.x86_64           1.0.2-2.el7.remi.7.2                @remi-php72
php-pecl-mysql.x86_64            1.0.0-0.17.20160812git230a828.el7.remi.7.2
                                                                     @remi-php72
php-pgsql.x86_64                 7.2.23-1.el7.remi                   @remi-php72
php-xml.x86_64                   7.2.23-1.el7.remi                   @remi-php72
php-xmlrpc.x86_64                7.2.23-1.el7.remi                   @remi-php72
php72-php-cli.x86_64             7.2.23-1.el7.remi                   @remi-safe 
php72-php-common.x86_64          7.2.23-1.el7.remi                   @remi-safe 
php72-php-json.x86_64            7.2.23-1.el7.remi                   @remi-safe 
php72-php-pdo.x86_64             7.2.23-1.el7.remi                   @remi-safe 
php72-php-pear.noarch            1:1.10.9-2.el7.remi                 @remi-safe 
php72-php-pgsql.x86_64           7.2.23-1.el7.remi                   @remi      
php72-php-process.x86_64         7.2.23-1.el7.remi                   @remi-safe 
php72-php-xml.x86_64             7.2.23-1.el7.remi                   @remi-safe 
php72-runtime.x86_64             2.0-1.el7.remi                      @remi-safe 
```

必要なのは、`php72-php-mbstring.x86_64`なのでは？ということで入れてみたところ、解決した。

```
# yum install php72-php-mbstring
```

```
$ sudo yum list installed | grep php
php.x86_64                       7.2.23-1.el7.remi                   @remi-php72
php-cli.x86_64                   7.2.23-1.el7.remi                   @remi-php72
php-common.x86_64                7.2.23-1.el7.remi                   @remi-php72
php-gd.x86_64                    7.2.23-1.el7.remi                   @remi-php72
php-json.x86_64                  7.2.23-1.el7.remi                   @remi-php72
php-mysqlnd.x86_64               7.2.23-1.el7.remi                   @remi-php72
php-pdo.x86_64                   7.2.23-1.el7.remi                   @remi-php72
php-pecl-mcrypt.x86_64           1.0.2-2.el7.remi.7.2                @remi-php72
php-pecl-mysql.x86_64            1.0.0-0.17.20160812git230a828.el7.remi.7.2
                                                                     @remi-php72
php-pgsql.x86_64                 7.2.23-1.el7.remi                   @remi-php72
php-xml.x86_64                   7.2.23-1.el7.remi                   @remi-php72
php-xmlrpc.x86_64                7.2.23-1.el7.remi                   @remi-php72
php72-php-cli.x86_64             7.2.23-1.el7.remi                   @remi-safe 
php72-php-common.x86_64          7.2.23-1.el7.remi                   @remi-safe 
php72-php-json.x86_64            7.2.23-1.el7.remi                   @remi-safe 
php72-php-mbstring.x86_64        7.2.23-1.el7.remi                   @remi-safe 
php72-php-pdo.x86_64             7.2.23-1.el7.remi                   @remi-safe 
php72-php-pear.noarch            1:1.10.9-2.el7.remi                 @remi-safe 
php72-php-pgsql.x86_64           7.2.23-1.el7.remi                   @remi      
php72-php-process.x86_64         7.2.23-1.el7.remi                   @remi-safe 
php72-php-xml.x86_64             7.2.23-1.el7.remi                   @remi-safe 
php72-runtime.x86_64             2.0-1.el7.remi                      @remi-safe
```

有効化されていることを確認！！

```
$ php --ini
Configuration File (php.ini) Path: /etc/opt/remi/php72
Loaded Configuration File:         /etc/opt/remi/php72/php.ini
Scan for additional .ini files in: /etc/opt/remi/php72/php.d
Additional .ini files parsed:      /etc/opt/remi/php72/php.d/20-bz2.ini,
/etc/opt/remi/php72/php.d/20-calendar.ini,
/etc/opt/remi/php72/php.d/20-ctype.ini,
/etc/opt/remi/php72/php.d/20-curl.ini,
/etc/opt/remi/php72/php.d/20-dom.ini,
/etc/opt/remi/php72/php.d/20-exif.ini,
/etc/opt/remi/php72/php.d/20-fileinfo.ini,
/etc/opt/remi/php72/php.d/20-ftp.ini,
/etc/opt/remi/php72/php.d/20-gettext.ini,
/etc/opt/remi/php72/php.d/20-iconv.ini,
/etc/opt/remi/php72/php.d/20-json.ini,
/etc/opt/remi/php72/php.d/20-mbstring.ini,
/etc/opt/remi/php72/php.d/20-pdo.ini,
/etc/opt/remi/php72/php.d/20-pgsql.ini,
/etc/opt/remi/php72/php.d/20-phar.ini,
/etc/opt/remi/php72/php.d/20-posix.ini,
/etc/opt/remi/php72/php.d/20-shmop.ini,
/etc/opt/remi/php72/php.d/20-simplexml.ini,
/etc/opt/remi/php72/php.d/20-sockets.ini,
/etc/opt/remi/php72/php.d/20-sqlite3.ini,
/etc/opt/remi/php72/php.d/20-sysvmsg.ini,
/etc/opt/remi/php72/php.d/20-sysvsem.ini,
/etc/opt/remi/php72/php.d/20-sysvshm.ini,
/etc/opt/remi/php72/php.d/20-tokenizer.ini,
/etc/opt/remi/php72/php.d/20-xml.ini,
/etc/opt/remi/php72/php.d/20-xmlwriter.ini,
/etc/opt/remi/php72/php.d/20-xsl.ini,
/etc/opt/remi/php72/php.d/30-pdo_pgsql.ini,
/etc/opt/remi/php72/php.d/30-pdo_sqlite.ini,
/etc/opt/remi/php72/php.d/30-wddx.ini,
/etc/opt/remi/php72/php.d/30-xmlreader.ini
```

# あとがき
諦めて一か月ほど放置してました(笑)
解決してよかった。
