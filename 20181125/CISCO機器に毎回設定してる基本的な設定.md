---
ID: b221ae650699938cb5dd
Title: CISCO機器に毎回設定してる基本的な設定
Tags: Cisco
Author: ""
Private: false
---

[C1721のInstallation Guide](http://perso.univ-lyon1.fr/olivier.gluck/Cours/Supports/L3IF_RE/CNA/CISCO1721/HardConfigGuide.pdf)

頭の中の整理を兼ねて。

3行でまとめると、
・毎度SSH接続を有効にして、
・デフォルトでは見にくい、ログ出力の設定を見やすいように調整する
ってことです。

#SSH接続を可能にする
・ユーザを作成する
・ドメインの設定
・ルータ用鍵ペアの生成
・SSH接続を有効にする

https://www.infraexpert.com/study/ciscorouter7.html

#loggingの設定を最適化する
・打刻の設定を見やすいものにする
（初期設定：ルータの起動時からの経過時間　→　日本時刻へ変更）
・ルータの時刻同期を設定し、正確な時間を刻ませる
（NTPサーバの設定など）
・logging synchronousをlineインターフェース、VTYインターフェースに設定する

https://www.infraexpert.com/study/syslog2.html
