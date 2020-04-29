---
ID: 16006f3622551e231a2e
Title: pipを直で使うのは非推奨っぽい？
Tags: Python,pip
Author: ""
Private: false
---

# エラーメッセージが出てました。

いつものように、新規に建てたインスタンスに`python3-pip`をインストールした後、`pip`を使おうとするとエラーメッセージが出ました。

```
$ pip3 install pip
WARNING: pip is being invoked by an old script wrapper. This will fail in a future version of pip.
Please see https://github.com/pypa/pip/issues/5599 for advice on fixing the underlying issue.
To avoid this problem you can invoke Python with '-m pip' instead of running pip directly.
```

["please see"と書いてあるリンク](https://github.com/pypa/pip/issues/5599)を見てみると、pipのアップグレード後に色々と問題が発生してるみたいです。

`--user`をつけてインストールした方が、不具合が発生しにくくなるとか色々ありつつも、一番確実な方法は`python -m pip`でインストールすることみたいです。


私はいつもpipは最新版にしてから使うので、いつものように`--upgrade`してみます。

```
$ python3 -m pip install --upgrade pip
Collecting pip
  Cache entry deserialization failed, entry ignored
  Using cached https://files.pythonhosted.org/packages/54/0c/d01aa759fdc501a58f431eb594a17495f15b88da142ce14b5845662c13f3/pip-20.0.2-py2.py3-none-any.whl
Installing collected packages: pip
Successfully installed pip-20.0.2
```
正常に、最新版へのバージョンアップができました。

```
$ python3 -m pip --version
pip 20.0.2 from /home/ubuntu/.local/lib/python3.6/site-packages/pip (python 3.6)
```

今後はこの方法で使おうかなと思います。

## 補足
> `pip` が Python に付属するのは 3.4 以降です。
https://docs.python.org/ja/3.6/installing/index.html

そのため、python3.4より前の場合は上記の方法は使えません。今までどおりpipをインストールして使いましょう。
