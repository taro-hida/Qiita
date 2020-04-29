---
ID: 3ce63b60c5e8d52c04b3
Title: 自分用ごった煮メモ：python編
Tags: Python,自分用メモ
Author: ""
Private: false
---

#競技プログラムにおける標準入力用
以下が詳しいが、よく使うものについて抜粋する
https://qiita.com/kyuna/items/8ee8916c2f4e36321a1c

```input.txt
3
3 1 2
2 5 4
3 6
```

```input.py

numberN = int(input())
seqAi = list(map(int,input().split()))
benefitBi = list(map(int,input().split()))
increaseCi = list(map(int,input().split()))
```

実行は、以下のようにすればOK

```
$ cat input.txt | input.py
```



#member関数から、同じclassのmember関数を呼び出すときは、引数にselfをつける

```
class SampleApi:

    def get_service_status(host=True, service=True, hogehoge=True):
        return self.__api_request(self, method, request_params)

    def __api_request(self, method=None, params=None):
        if not isinstance(method, str):
            print('error: arg[method] must be string')
            print(method)
            return False
```
これで実行すると、methodにparamsが入って
`error: arg[method] must be string`と言われる。

selfを引数に指定しましょう



#左側を0でパディングした日付を生成したい
左側を0でパディングした日付を生成するのに
`datetime` moduleの、`strftime()`
が非常に便利。

```
hidaro@debean3:~$ python  
Python 2.7.3 (default, Nov 24 2017, 16:26:37) 
[GCC 4.7.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import datetime
>>> dt_now = datetime.datetime.now
>>> print(dt_now.strftime('%H'))
08
>>> print(dt_now.strftime('%M'))
09
>>> print(dt_now.strftime('%m\%2F'))
09\2018-09-11
>>> print(dt_now.strftime('%m%%2F'))
09%2F
>>> print(dt_now.strftime('%Y%%2F%m%%2F%D'))
2018%2F09%2F09/11/18
>>> print(dt_now.strftime('%Y%%2F%m%%2F%d'))
2018%2F09%2F11
```

%を出力したい場合は`%%`でいいみたいです。
公式ドキュメント -> https://docs.python.org/ja/3/library/datetime.html#strftime-strptime-behavior

#append, extendの違い

https://www.pythonweb.jp/tutorial/list/index6.html

```
list = ["A", "B", "C"]

list.append(["D", "E"])
print list      # ["A", "B", "C", ["D", "E"]]

list.extend(["D", "E"])
rint list      # ["A", "B", "C", "D", "E"]
```

よく忘れるので...
