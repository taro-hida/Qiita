---
ID: a4f25ec0bfebd74c12da
Title: maillogから、送信においてなんらかのエラーが発生したログをgrepするコマンド
Tags: postfix,maillog
Author: ""
Private: false
---

```
less /var/log/maillog |grep -v 'dsn=2\.0\.0'  | grep 'dsn=' 
```

SMTPは、正常な送信を行うと`dsn=2.0.0`を返しますので、それを`grep -v`で除外してやると、なんらかのエラーが起こったあたりを抜き出すことができるのではないか。

dsnについては以下参照。httpのステータスコードみたいなもの
https://oxynotes.com/?p=9810

```
less /var/log/maillog |grep -v 'dsn=2\.0\.0' | grep 'dsn=' -A 4
```
のようにすれば、その行だけでなく、該当した後の数行（この場合は4行）も見れてみやすい

ちなみに、とりあえず送信できていないやつだけgrepしたい場合（エラーが出てても、送信済みなら問題ないとする場合）は、status=sentの方をgrepしてやると見やすいと思う。

```
less /var/log/maillog | grep -v 'status=sent' | grep 'status=' -A 4
```
みたいな感じで。

なにはともあれ、-v オプションと　-Aオプションは便利。
