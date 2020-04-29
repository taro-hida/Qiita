---
ID: bbb3ed18fbb8b1e4dd48
Title: Powershellを使って ` du -sh | head -n 20`をする
Tags: PowerShell
Author: ""
Private: false
---

# コマンド
```PowerShell
(Get-ChildItem -Path C:\ -Recurse | Select-Object FullName,  Length | Sort-Object Length -Descending)[0..20]
```
- 上記コマンドはCドライブ直下のファイル全てを探すので、powershellは管理者として発行する

# 参考
[https://nasrinjp1.hatenablog.com/entry/2017/01/15/232620](https://nasrinjp1.hatenablog.com/entry/2017/01/15/232620)
[https://orebibou.com/2015/01/PowerShellでhead,tail相当の処理を行う/](https://orebibou.com/2015/01/PowerShellでhead,tail相当の処理を行う/)
