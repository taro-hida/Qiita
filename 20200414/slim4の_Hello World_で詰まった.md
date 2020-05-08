---
ID: a4f47d6bca2ed4063efb
Title: slim4の"Hello World"で詰まった
Tags: Slim4
Author: ""
Private: false
---

# Step 4: Hello World
参考にしたのはこの公式ドキュメント
https://www.slimframework.com/docs/v4/start/installation.html

# 出たエラー
- "http://127.0.0.1/public/index.php" にアクセスすると、500のHTTP Error
 ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/124948/b9f8b8d8-8244-790b-cb2f-dd94c1abec3b.png)

- エラーログを確認すると、以下のメッセージ

```
$ less /var/log/apache2/error.log
PHP Fatal error:  Uncaught Slim\\Exception\\HttpNotFoundException: Not found.
```

全文

```
$ less /var/log/apache2/error.log
[Mon Apr 13 08:14:45.967574 2020] [php7:error] [pid 12654] [client ::1:34874] PHP Fatal error:  Uncaught Slim\\Exception\\HttpNotFoundException: Not found. in /home/taro/www-data/vendor/slim/slim/Slim/Middleware/RoutingMiddleware.php:93\nStack trace:\n#0 /home/taro/www-data/vendor/slim/slim/Slim/Routing/RouteRunner.php(72): Slim\\Middleware\\RoutingMiddleware->performRouting(Object(Slim\\Psr7\\Request))\n#1 /home/taro/www-data/vendor/slim/slim/Slim/MiddlewareDispatcher.php(81): Slim\\Routing\\RouteRunner->handle(Object(Slim\\Psr7\\Request))\n#2 /home/taro/www-data/vendor/slim/slim/Slim/App.php(211): Slim\\MiddlewareDispatcher->handle(Object(Slim\\Psr7\\Request))\n#3 /home/taro/www-data/vendor/slim/slim/Slim/App.php(195): Slim\\App->handle(Object(Slim\\Psr7\\Request))\n#4 /home/taro/www-data/index.php(15): Slim\\App->run()\n#5 {main}\n  thrown in /home/taro/www-data/vendor/slim/slim/Slim/Middleware/RoutingMiddleware.php on line 93
```

# 何が悪かったか
サンプルスクリプトの10 ~ 13行目の通り、サンプルスクリプトは`/`のpathでレスポンスを待ち受けているので、"http://127.0.0.1/public/index.php" のpathではなく、"http://127.0.0.1/" のpathでアクセスする必要がある。

- 10~13行目

```php
$app->get('/', function (Request $request, Response $response, $args) {
    $response->getBody()->write("Hello world!");
    return $response;
});
```

# 解決策1
1. 以下のように"index.php"のpathを変更する
2. "index.php" の `require_once`のpathを変更する
3. "http://127.0.0.1/" にアクセスする。

```
$ ls -ltr
total 36
-rw-r--r-- 1 root www-data    17  3月 28 22:15 phpinfo.php
-rw-r--r-- 1 taro www-data    78  4月 13 08:17 composer.json
-rw-r--r-- 1 taro www-data 20193  4月 13 08:17 composer.lock
drwxr-sr-x 8 taro www-data  4096  4月 13 08:17 vendor
drwxr-sr-x 2 taro www-data  4096  4月 14 08:17 public

$ mv public/index.php ./
$ ls -ltr
total 40
-rw-r--r-- 1 root www-data    17  3月 28 22:15 phpinfo.php
-rw-r--r-- 1 taro www-data   378  4月 13 08:14 index.php
-rw-r--r-- 1 taro www-data    78  4月 13 08:17 composer.json
-rw-r--r-- 1 taro www-data 20193  4月 13 08:17 composer.lock
drwxr-sr-x 8 taro www-data  4096  4月 13 08:17 vendor
drwxr-sr-x 2 taro www-data  4096  4月 14 08:26 public
```

```diff
- require __DIR__ . '/../vendor/autoload.php';
+ require __DIR__ . '/vendor/autoload.php';
```

# 解決策2
1. DocumentRootを変更する
2. apache2をreloadして変更を反映する
3. "http://127.0.0.1/" にアクセスする。

```
$ sudo vi /etc/apache2/sites-enabled/000-default.conf
```

```diff
- DocumentRoot /home/taro/www-data
+ DocumentRoot /home/taro/www-data/public
```

```
$ systemctl apache2 reload
```

# 環境
Ubuntu 18.04
apache2

# おわりに
なんか変なエラーログが出たので、それを解決しようと変な方向に努力してしまった。

https://qiita.com/ryu19maki/items/10518cb1e306e213c2ac でやってるみたいに、

```
$ php -S localhost:8080 -t public
```
ってphpにコンテンツをServeさせた方が良かったのかもしれない。
