---
ID: 0c200693ab92dbccadfa
Title: ubuntu18.04 への apache2 新規設定時の手順
Tags: apache2,自分用メモ,ubuntu18.04
Author: ""
Private: false
---

# 発行コマンド
環境を作成するたび調べることになるので自分用に

```
id taro
sudo usermod -aG www-data taro 
id taro 
mkdir ~taro/www-data
sudo chown -R taro.www-data ./www-data/
sudo chmod -R 2770 ./www-data/
sudo find ./www-data/ -type d -exec chmod 770 {} \;
sudo find ./www-data/ -type f -exec chmod 660 {} \;
sudo sed -i".org" -r "s/DocumentRoot.*$/DocumentRoot \/home\/taro\/www-data/" /etc/apache2/sites-enabled/000-default.conf 
sudo vim /etc/apache2/apache2.conf
```

# 何をしているか
```
# グループ事前確認
id taro
sudo usermod -aG www-data taro 
# 変更反映の確認
id taro 
mkdir ~taro/www-data
sudo chown -R taro.www-data ./www-data/
# SGIDを設定する(https://eng-entrance.com/linux-permission-sgid)
sudo chmod -R 2770 ./www-data/
# 既存のディレクトリ、ファイルのパーミッションを置き換える
sudo find ./www-data/ -type d -exec chmod 770 {} \;
sudo find ./www-data/ -type f -exec chmod 660 {} \;
# sedで書き換える（趣味）。バックアップも作成
sudo sed -i".org" -r "s/DocumentRoot.*$/DocumentRoot \/home\/taro\/www-data/" /etc/apache2/sites-enabled/000-default.conf
# directoryに許可設定を入れる(内容については後述)
sudo vim /etc/apache2/apache2.conf
```

`/etc/apache2/apache2.conf`については、以下の設定を追加しないといけなかった（詰まった）。

```
<Directory /home/taro/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
```

# 知見
`~taro/www-data/`という形式でディレクトリを指定すると403が出る。
(`apache2.conf`でも、`000-default.conf`でも)

\<Directory\> 〜 \</Directory\>で使えるpathは、full pathか正規表現だけみたいですね。
> Directory-path is either the full path to a directory, or a wild-card string using Unix shell-style matching. In a wild-card string, ? matches any single character, and * matches any sequences of characters. 

> Regular expressions can also be used, with the addition of the ~ character. For example:

ref: https://httpd.apache.org/docs/2.4/en/mod/core.html#directory

# 補足
phpを使うときは

```
$ sudo apt install libapache2-mod-php7.2
```

# 環境

最近使っているノートPCにインストールしたUbuntu18.04

```
$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="18.04.4 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.4 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```
- apache2

```
$ sudo apache2 -V
Server version: Apache/2.4.29 (Ubuntu)
Server built:   2020-03-13T12:26:16
Server's Module Magic Number: 20120211:68
Server loaded:  APR 1.6.3, APR-UTIL 1.6.1
Compiled using: APR 1.6.3, APR-UTIL 1.6.1
Architecture:   64-bit
```

# 参考
https://freelance.techcareer.jp/features/16
https://askubuntu.com/questions/325498/apache-cant-access-folders-in-my-home-directory
https://open-groove.net/linux-command/sed-edit-files/
