---
ID: f677abe2bc3b689002b3
Title: 'heroku環境でmb_xxx関数を使うと、Fatal error: Call to undefined function …のエラーが表示される'
Tags: PHP,Heroku,mbstring
Author: ""
Private: false
---

Fatal error: Call to undefined function …っていうのは、
「深刻なエラーです：未定義の関数を呼び出しています」ってことですね。
このエラーが出る場合によくあるのが、マルチバイト関数が無効化されている状態でマルチバイト関数を呼び出しているケースです。

マルチバイト関数というのは、マルチバイト文字（日本語など）用の関数群で、mb_strpos()、mb_substr()、mb_strlen()などがあります。

特に、herokuのphp環境は、マルチバイト関数がデフォルトで無効の設定になっているので、どっかで調べてきて使おうとしてもエラーが表示されて使えません。

ちなみに、ローカル環境でしたら、php.iniのコメントアウトを消去することでマルチバイト関数を有効にします。

```php.ini
;extension=php_mbstring.dll
```

この行のコメントアウト、";"を取り除けば、マルチバイト関数が有効になります。

しかし、heroku環境ではデプロイ環境のphp.iniをいじることができません。
さて本題です。
herokuでマルチバイト関数を有効にするには、composerに"ext-mbstring"を追加してPushします。

```composer.json
"require": {
  "ext-mbstring": "*"
},
```

こんな感じ。
あと、忘れてはいけないのがcomposer.lockの更新です。
CLI（Windowsならコマンドプロンプト）を開いて、アプリケーションのFile上で、composer updateコマンドを叩きます。

```コマンドプロンプト.
...\heroku\アプリ名> composer update
```

私はこれに気づかずしばらくうんうん言ってましたｗ

以上の手順を踏めば、マルチバイト関数が使えるようになるはずになり、原因が正しい場合は、エラーも解決されるはずです。

参考：
[Heroku上でmbstringなどcomposerを用いて、有効にする方法-雄大スタジアム](http://yudai-stadium.com/blog/heroku-mbstring/)
[「Fatal error: Call to undefined function …」と表示されるとき-PHPプログラミングの教科書 [php1st.com]](https://php1st.com/419/)
