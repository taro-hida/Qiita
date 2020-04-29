---
ID: 384cc8dd156364cb035f
Title: shellscriptを使って、sshの引数にtab補完を効かせる
Tags: shell,complete
Author: ""
Private: false
---

https://unix.stackexchange.com/questions/11376/what-does-double-dash-mean-also-known-as-bare-double-dash

ぶっちゃけ上の記事で全部事足ります。少し難しいけど。

#やりたいこと

自分の環境を晒すわけにもいかないので、例えの話。

ip: 192.168.0.1のホスト[hogehost]と、
ip: 192.168.4.2のホスト[hugahost]があるとします。

いちいち `ssh username@192.168.0.1`と打つのは面倒です。
`ssh hogehost`と打ったら、勝手に`ssh username@192.168.0.1`とコマンドを発行してほしい！！！

これに関しては、`.ssh/config`というファイルを設定することで達成することができました。
詳しくは検索してみてくださいね。
https://webkaru.net/linux/ssh-config-file/
この記事、簡潔でわかりやすかったですね。

さて、次は、、、

`ssh <tab>`と打った時に、host名で補完を効かせたい！

```
ssh <tab>
 hogehosy hugahost
```

ずばりこうしたい！！

#どうやるか
-> [説明要らない人はこっち](https://qiita.com/hidaro/items/384cc8dd156364cb035f#%E3%81%9D%E3%81%AE%E7%B5%90%E6%9E%9C%E3%81%8C%E3%81%93%E3%81%A1%E3%82%89%E3%81%AB%E3%81%AA%E3%82%8A%E3%81%BE%E3%81%99)

せっかく.ssh/configを作ったので、これを使いましょう。
うれしいことに、きれいな規則に沿って並んでくれてます。

先ほどの記事の一部を引用すると

>\#Aサーバー
Host sakura
  HostName aaa.bbb.ccc.ddd
  Port 22
  User root

これですね。

この例で言うと、 'sakura'の部分が欲しいです。.ssh/configのおかげで、
`ssh sakura`と打つと`ssh root@aaa.bbb.ccc.ddd:22`と発行してくれます。

では、まずgrepで'^Host 'とついてる行を抜き出しちゃいましょう。
`grep '^Host '  ~/.ssh/config`
そして、その結果をawkコマンドで出力します。
`grep '^Host ' ~/.ssh/config | awk '{print $2}'`

これでsakuraがとれたはず。

んじゃあ、それを関数化して...

```
ssh_get () {
  COMPREPLY=(`grep '^Host ' ~/.ssh/config | awk '{print $2}'`)

complete -F ssh_get ssh
```

こんな感じに.bashrcにぶち込み（追記し）ます（忘れずに `source .bashrc`発行して読み込みなおしてね）

そうすると、`ssh <tab>`とするたびに .ssh/configに記載されたホストが表示されるはずです！感動！

しかし、このままでは`ssh s<tab>`としても、`ssh a<tab>`としても全てのホストが表示されてしまいます。

そうじゃない！僕らは`ssh s<tab>`と押したら

```
sakura saku_host sakana
```
みたいに補完してくれるタブ補完を求めている！！

まあ、
`grep '^Host ' ~/.ssh/config | awk '{print $2}'`
にgrepをかければいいんですが、

`ssh s<tab>`としたときの's'が、どのような形で取得できるのかが謎でした。

そこで一番最初の記事。
https://unix.stackexchange.com/questions/11376/what-does-double-dash-mean-also-known-as-bare-double-dash

そして、
<b>`${COMP_WORDS[COMP_CWORD]}`</b>

ずばりこれです。
これでgrepすればいけるわけですね。

`grep '^Host ' ~/.ssh/config | awk '{print $2}' | grep '^'$curw`

これをCOMPREPLY=()の中にぶち込みましょう。

#####その結果がこちらになります。

```
ssh_get_m () {
  local curw
  curw=${COMP_WORDS[COMP_CWORD]}
#COMPREPLY=($(compgen -W `grep '^Host ' ~/.ssh/config | awk '{print $2}'` -- $curw))
  COMPREPLY=(`grep '^Host ' ~/.ssh/config | awk '{print $2}' | grep '^'$curw`)
}
complete -F ssh_get_m ssh
```

これを.bashrcに追記して読み込んでやれば、.ssh/configに記載さえしていれば補完を効かせることができます。
コメントアウトしている部分は、冒頭の記事で紹介されていた方法です。compgenという関数があるので、それを利用しています。

自分の中での整理もかねて、少し丁寧目に説明しましたが、上の関数をそのまま使っていただければ無事に動くと思います。（.ssh/configがちゃんと記載されていれば）
ただ、できればぜひ自分で理解したうえで使ってくださいね。

次はこれに続けてパスワード入力も自動でしてもらうようにするのが目標。だれかいい記事教えてくんろ...

-> しました。
https://qiita.com/hidaro/items/d65079adcfc44511e3bf
