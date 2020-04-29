---
ID: ec00926b68eb74d0d0e8
Title: PHP関連の覚書
Tags: PHP,自分用メモ
Author: ""
Private: false
---


#phpが400番、500番のエラー吐いた時(INTERNAL SERVER ERRORとか)に、Apacheのエラーログを見て原因をたどっていく方法【CentOS7】

これまじで便利なのでおすすめ。

Apacheのエラーログは、
`/etc/httpd/logs`以下にあります（デフォルト設定）

基本的にエラーログは、error_log [-20180521] ※日付があったりなかったり
って形で保存されてますので、`less`とか`tail`を使って閲覧します。

私のように、https接続を行っている場合、https通信に関してのerror_logは
`ssl_error_log`ファイルに書き込まれており、`error_log`には残らないのでその点だけ注意です！

#phpで変数の中身をファイルに出力(05/24)

```php
int file_put_contents ( string $filename , mixed $data);
```
が便利。

ただし、配列を出力する場合、これの`$data`に配列を入れても
`Array`
とだけ出力されて肝心のファイルの中身が表示されない。

`print_r()`関数を使うと、返り値としてわかりやすい形式が返るので、

```php
$dump = print_r( $array , TRUE);
$file_put_contents('hogefile', $dump);
```

とすれば、

```
Array
(
    [events] => Array
        (
            [0] => Array
                (
                    [type] => message
                    [replyToken] => xxxxxxxx
                    [source] => Array
                        (
                            [userId] => xxxxxxxx
                            [type] => user
                        )

                    [timestamp] => xxxxxxxxx
                    [message] => Array
                        (
                            [type] => text
                            [id] => xxxxxxx
                            [text] => hoge
                        )

                )
        )
)

```
みたいな見慣れた形式の出力が指定したhogefileに書き込まれているはず。
ちなみに、hogefileが存在しない場合勝手に作成してくれます。

[参考元](https://teratail.com/questions/5580)


#型チェック
使い方は簡単。gettype()関数を使うだけ。

```php
echo gettype($ony_total[0]);

-> string
```
preg_matchの第三引数の型を確かめるときに使用。デバッグに役立つ。
参考：公式ドキュメント

#phpの日付

date()関数を使う。文字列を中に入れて

```php

echo $date("m月d日");

-> 04月24日

echo $date("n月j日");

-> 4月24日
```

ご覧の通り、0を挿入して2桁にしてくれるパターンと、そのまま出力してくれるパターンがあります。

参考にしたのはこれ：[phpの日付についての記事](http://raining.bear-life.com/php/php%E3%81%AEdate%E9%96%A2%E6%95%B0%E3%81%A7%E6%97%A5%E4%BB%98%E3%81%AE%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%83%E3%83%88%E5%A4%89%E6%9B%B4)
正直これ見たら十分
#phpの正規表現(04/24)

```php

$text = "5回";
preg_match("/^[0-9]+/", $text, $array);
echo $array[0];

-> 5
```
####ポイント
・第三引数には、配列が返る。$array[0]には、マッチした分の文字列が入る。
・<b>第三引数の$arrayは、事前に宣言されてなくてもよい</b>
・文字列型で返るので、数字を抽出する場合は(int)によるキャストが必要。
参考：公式ドキュメント
