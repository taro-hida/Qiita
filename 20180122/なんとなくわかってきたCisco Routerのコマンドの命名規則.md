---
ID: 177c2320b2ddbb91846d
Title: なんとなくわかってきたCisco Routerのコマンドの命名規則
Tags: command,Cisco,CCNA
Author: ""
Private: false
---

なんとなくCiscoRouterのコマンドの命名規則がわかってきたので、ここに記そうと思います。

単語カードなどを使ってバーッと覚える方法もいいと思うんですが、それを異様に苦痛に感じる人種がいます。~~私です~~。
この方法を使えばある程度整理して覚えられますので、ぜひ有効活用してみてください。私も適宜追加していきます。

そしてベテランの皆様。私はCCNAべんきょーちゅうの見習いです、間違っているところがあったら指摘してください。
加えて、当記事は階層表として不完全です。当記事を元にしていただいても全然かまいませんので、完全な階層表やその解説記事を書いていただけると助かります...＿( \_´ω`)\_ていうかむしろぜひお願いします...


さて、
CiscoRouterのコマンドは、ある程度の階層構造になっています。
一番わかりやすいのは`ip`。
`(config)#ip address 192.168.1.0 255.255.255.0`とか、
`(config)#ip rip`とか、
`(config)#ip dhcp`とか。
ipと頭についてるのは、すべてip(internet protocol)関連のものです。ちょっと大雑把ですが、TCP/IPモデルの第2層。OSIモデルの第3層と言い換えてもいい。
ユーザモード(>)や特権モード(execモード,\#)で使用するshowコマンドに続くip関連のコマンドも、同様の規則があります。
`>show ip route`とか、
`#show ip int brief`とかよく使うのではないでしょうか。
また、debugコマンド打つ時も。
`#debug ip nat`
`#debug ip eigrp`
みたいになりますね。

以下、ip関連のコマンド一覧を表示します。解説が必要だと思ったところは下に解説入れてます。この命令規則でいくと、access-list辺りで混乱するんですが、自分なりの解釈も入れてますので是非そこは読んでみてください。

```
ip address ...
     ip address dhcp
ip dhcp ...
     ip dhcp pool
     ip dhcp excluded-address
     ip default-gateway
     ip helper-address 
     => routerがdhcp discoverをdhcp serverへ中継するための設定
ip nat ...
     ip nat inside sourse ...
     ip nat inside
     ip nat outside
     show ip dhcp pool
     show ip nat translations
     clear ip nat translations
     debug ip nat
ip route ...
ip access-list ...
     ip access-list {standard | extended} ...
     ip access-group
```
※アクセスリスト関係に関しては、
(config)#access-class => vtyインターフェースにACL設定
        #show access-list => ipx等含めたACL表示
(config)#access-list {番号|名前} => 番号付きACL設定
の例外があるので注意を

TCP/IPのInternet層にはipのほかに、ipxなどのプロトコルがあります。名前付きACLのコマンドはipのみ対応してるのでipが頭につく一方で、番号付きACLのコマンドはipxなどほかのプロトコルにも対応しているため、頭にipがつかないのだろうと解釈しています。
showコマンドを打つ際にも、show access-listとshow ip access-listのコマンドがあり、ipを付けてない場合だとすべてのACLが、ipを付けた場合だとipプロトコル上のACLのみが表示されます。

```
ip host：ローカルでドメイン⇔ipの変換をするhost fileについてのコマンドです。ipに関連してますね。
ip ospf cost
ip eigrp...
show ip route [ ospf | bgp ]
show ip arp
show ip protocols：ipに関するプロトコルを表示します
show ip ospf ...
show ip eigrp...
show ip bgp ...
clear ip ospf process
```

また、
`(config)#ip rip`
などのコマンド自体は`ip`階下のコマンドなんですが、打った後に入る`(config-router)#`で打つ
`(config-router)#rip v2`や
`(config-router)#network`
`(config-router)#default-information originate`
などのコマンドはip階下とならないので注意してください。
同様に、
`(config)#ip dhcp pool pool1`
のコマンドはip階下ですが、
`(dhcp-config)#network 192.168.1.0 255.255.255.0`
`(dhcp-config)#default-router 192.168.1.1`
のように、`(dhcp-config)#`に入ると頭に`ip`がつかなくなります。
この法則はいろんなところで表れます。

#まとめ
ip（っていうプロトコル）に関連するものは頭にipがつく
ipxとかのほかのプロトコルが関連してくるとipがつかなくなる
ルータコンフィグレーションモードなどに入るとipがつかなくなる
