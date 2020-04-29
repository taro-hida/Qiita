---
ID: 3f73c0b63ceb0a79cf59
Title: CentOS7でPDO使ってpostgresにつないでたら、"Ident authentication failed for user"ってエラーログが出た
Tags: PHP,PostgreSQL,PDO
Author: ""
Private: false
---

CentOS7でPDOを使ってpostgresqlにつないでたら、

```
[Sun Jul 01 18:28:19.399178 2018] [:error] [pid 2883] [client xxx.xxx.xxx.xxx] PHP Fatal error:  Uncaught exception 'PDOException' with message 'SQLSTATE[08006] [7] FATAL:  Ident authentication failed for user "hogeuser"' in ...
```

っていう内容がエラーログに吐かれていました。

読んでの通り、「hogeuserってユーザ（postgresqlのユーザ）の認証が失敗しましたってことなんですが、

```php
$db = new PDO("pgsql:dbname=hogedb;host=localhost;", "hogeuser", "hogepassword");
```

その時のコードはこの通り。
PHP公式ページの接続例とほとんど相違ありません
接続例url->
[http://php.net/manual/ja/pdo.connections.php](http://php.net/manual/ja/pdo.connections.php)

こんな時はググってみようということで、エラーメッセージでググる。

Qiitaにこんな記事↓
[https://qiita.com/pugiemonn/items/7ec47bc82bd56b0458b9](https://qiita.com/pugiemonn/items/7ec47bc82bd56b0458b9)

このブログで一つの解決策が
`/var/lib/pgsql/9.5/data/pg_hba.conf`っていうファイルを書き換えればいいらしい。
でもこれは、`hogeuser`のパスワード認証を無効にするということ。
公開を目的としていないプログラムならいいが、公開を目的とするプログラムには適さない。

ベストソリューションはこちら。
https://blog.fkoji.com/2015/02260627.html

内容としては、
<b>`host=localhost`の指定を外す</b>
それだけ。

PDOからインスタンスを生成するときに渡すdsnにおいて、hostを指定しない場合はデフォルトでlocalhostが指定されるので、外しても問題なし。

というわけで、さっきのコードからhost=localhostを消してみる。


```php
$db = new PDO("pgsql:dbname=hogedb;", "hogeuser", "hogepassword");
```

無事エラーを吐かず接続。select文とinsert文が実行できたことを確認。

パスワード認証をしたのはこれで初めてだったが、これにて一件落着！

同じようなエラーが出た場合は、ぜひ試してみてください。
