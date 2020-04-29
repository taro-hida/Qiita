---
ID: 4b9a136cc82e579ff4df
Title: wmicでサービスを再起動する
Tags: windows2016,wmic
Author: ""
Private: false
---

# 発行コマンド

### 自ホストのサービスを起動・停止する

```console
# サービスを起動する
> wmic service SNMP call startservice

# サービスを停止する
> wmic service SNMP call startservice
```

### リモートホストのサービスを起動・停止する

```console
# サービスを起動する
> wmic /NODE:192.168.0.1 /User:Administrator service SNMP call stopservice

# サービスを停止する
> wmic /NODE:192.168.0.1 /User:Administrator service SNMP call startservice
```

今回はSNMPを指定しているが、他のサービスを操作したいときは、

```console
> wmic service list instance
Name
Browser
Dhcp
SNMP
EventLog
...(略
```
と表示してくれるので、操作したいサービス名に書き換えてコマンドを発行する。

# 返り値

返り値として数字が表示される。
start（起動）して`0`が返ってきたら成功、`10`が返ってきたら既に起動しているという意味。
詳しくは[ここ](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/startservice-method-in-class-win32-service)を参照。

stop（停止）して`0`が返ってきたら成功、すでに停止していると`5`が返ってくる。
詳しくは[ここ](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/stopservice-method-in-class-win32-service)を参照。

# 参考
[ネタ元](https://www.oreilly.com/library/view/windows-server-cookbook/0596006330/ch07s02.html)
[先人 - 別端末(Windows)のプログラムを標準機能でリモート起動する方法まとめ](https://qiita.com/0829/items/5518256b348521ac358c)

