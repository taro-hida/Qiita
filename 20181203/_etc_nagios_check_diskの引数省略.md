---
ID: 9ade5361fc3898704a53
Title: /etc/nagios/check_diskの引数省略
Tags: Nagios,nagios-nrpe
Author: ""
Private: false
---

#???

以下の項目がわからなかった

```/etc/nagios/nrpe.cfg
command[check_disk]=/usr/lib64/nagios/plugins/check_disk 80 95
```

/etc/nagios/check_diskにおいて、対象フォルダの引数がない時は、
各パーティション各々を対象として、チェックを実行する。

```
[root@t-tona ~]# df -Th
ファイルシス   タイプ   サイズ  使用  残り 使用% マウント位置
/dev/vda2      ext4        20G  7.7G   11G   42% /
devtmpfs       devtmpfs   234M     0  234M    0% /dev
tmpfs          tmpfs      244M     0  244M    0% /dev/shm
tmpfs          tmpfs      244M   17M  228M    7% /run
tmpfs          tmpfs      244M     0  244M    0% /sys/fs/cgroup
tmpfs          tmpfs       49M     0   49M    0% /run/user/1000
```
これの各々。tmpfsについてはわからない

引数が数字のみで指定されている場合は、使用している量の%表示を表すので、
今回の設定だと、80%以上の時にWarning, 95%の時にCritical

#...
ややこしいからちゃんと -w とか -c とかつけて書いてよ！！！
