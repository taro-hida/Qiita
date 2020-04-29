---
ID: 5c1df6e0839baea85e6b
Title: Cisco Routerにおける、OSPF設定の手順
Tags: Cisco,Router,OSPF
Author: ""
Private: false
---

![cap.PNG](https://qiita-image-store.s3.amazonaws.com/0/124948/e358e9d6-26bc-5a0d-f9ed-91e60a78a67c.png)
こんな感じのネットワークを想定します。
Cisco Packet tracerにて作成。
前提として
○routerの各interfaceはip address 割り当て済み
○routerの各interfaceにno shutdownコマンド入力済み
○各ホストにはdhcpでip addressを割り当て済み。default gatewayも割り当て済み

この状態から、OSPFを用いてroutingを行う場合は、各routerに以下のコマンドを入力します。

```
>enable
#conf t
(config)#router ospf 1
//ospfプロセスを有効化するコマンド。1はprocess-idであり、一つのrouter上で複数のOSPFプロセスを実行する際に使用されます。

(config-router)#network 192.168.1.0 0.0.0.255 area 1
//ospfを有効にするinterfaceをネットワークアドレスによって指定するコマンド。0.0.0.255はワイルドカードマスクです。area 1 はospfのarea指定。
```

以上のコマンドをそれぞれのrouterに打ち込めば、OSPFでのroutingができて、host同士の通信が行えるはずです。
