---
ID: ea44a002547a5de2596d
Title: Ubuntu18.04 LTSでgitの最新版をインストールしたい場合
Tags: Ubuntu
Author: ""
Private: false
---

# 環境
Ubuntu18.04 LTS

# 参照する手順
デフォルトで入っているレポジトリでは、古いバージョンが提供されていることが多い。
`sudo apt-cache madison git`を発行することで、利用可能なバージョンが確認できる

```
$ sudo apt-cache madison git
       git | 1:2.17.1-1ubuntu0.6 | http://jp.archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages
       git | 1:2.17.1-1ubuntu0.6 | http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages
       git | 1:2.17.0-1ubuntu1 | http://jp.archive.ubuntu.com/ubuntu bionic/main amd64 Packages
```

"git install documentation" とかで検索すると以下のページが引っかかるが、
https://git-scm.com/book/ja/v2/使い始める-Gitのインストール

package managerで管理したいときは、以下の手順を参照してレポジトリを追加する
https://git-scm.com/download/linux

# 確認

```
$ apt-cache madison git
       git | 1:2.26.1-0ppa1~ubuntu18.04.1 | http://ppa.launchpad.net/git-core/ppa/ubuntu bionic/main amd64 Packages
       git | 1:2.17.1-1ubuntu0.6 | http://jp.archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages
       ...
```

利用可能なバージョンが増えたことが確認できた。
