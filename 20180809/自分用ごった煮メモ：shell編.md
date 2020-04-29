---
ID: bc3a48a4b7ff105b7f08
Title: 自分用ごった煮メモ：shell編
Tags: shell,自分用メモ
Author: ""
Private: false
---

自分用のメモです。
ネットに転がってるものを主にまとめたもの。おそらく自分以外が読んでもあまり意味のない内容になるとおもいますので、ご了承ください。

#shellのpathについて
- pathは`:`で区切られる
- pathを追加する際は、`PATH=$PATH:/home/user/mybin`みたいな形で行う。<b>`:`</b>で区切るのがポイント。<br>pythonのpath追加に関しても同様・<br>`export PYTHONPATH=$PYTHONPATH:/home/user/python/module`を、<br>`.bashrc`に書き込んどきましょう。
