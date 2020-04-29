---
ID: d7066c85a1c55af2068b
Title: nekoサブドメインでにゃーんと返す実装をする
Tags: PHP
Author: ""
Private: false
---

# 何をしたいか
example.comのドメインを所有していた場合、neko.example.comに対するアクセスに"にゃーん"を応答する。
APIっぽく実装する。

今回使う僕のドメインは、taro-hida.tk

# 事前準備
- DNSで、neko.taro-hida.tkのサブドメインにAレコードを設定する。conohaのDNSで設定。

```
$ dig neko.taro-hida.tk +short
150.95.153.122
```
- Apacheサーバで、neko.taro-hida.tkのvirtualhostの設定を書く。

```
<VirtualHost *:80>
ServerName neko.taro-hida.tk 
DocumentRoot /var/www/neko/
CustomLog "logs/access_log" combined
ErrorLog "logs/error_log"
</VirtualHost>
```
# APIの実装
- Documentルートに`index.php`を設置する[^1]
[^1]: [PHPで簡単なWebAPIを実装してみる](https://note.com/kazztech/n/ndb3a5468f299)のコードを拝借

一応、index.phpについては権限をapacheに与えておく。

```
$ sudo chown apache:apache /var/www/neko/index.php
```

```php
# cat /var/www/neko/index.php
<?php
header('Content-Type: application/json; charset=utf-8');
$neko = array('にゃーん');

print json_encode($neko, JSON_PRETTY_PRINT);
```

# httpクエリを投げてjsonを取得
ブラウザでアクセスしても、json_encodeされたデータのため、UTF-8でencodeされたjsonがそのまま表示されてしまいます。
以下のpythonスクリプトでデータの取得を行います。

```python
# cat get.py 
import requests
import json

url = 'http://neko.taro-hida.tk/'

response = requests.get(url)
contents = json.loads(response.text)

print(contents)
```

```
$ python3 get.py 
['にゃーん']
```

望んでいた結果が得られました。

# おわりに
思い付きの実装でしたが、API実装したの初めてだったので勉強になったように思います。
割と30分くらいで実装もできました。

nekoサブドメインへのにゃーんAPI設置、流行らないかなぁ...

# 参考
- [PHPで簡単なWebAPIを実装してみる](https://note.com/kazztech/n/ndb3a5468f299)
- [Requestsで日本語を扱うときの文字化けを直す](https://qiita.com/nittyan/items/d3f49a7699296a58605b)
- [nkmkさんのページ](https://note.nkmk.me/python-json-load-dump/)
- [requestモジュールの公式ドキュメント](https://requests-docs-ja.readthedocs.io/en/latest/user/quickstart/)

