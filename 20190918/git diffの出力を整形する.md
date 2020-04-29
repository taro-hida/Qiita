---
ID: 64de6e69d4e0915c45c5
Title: git diffの出力を整形する
Tags: ShellScript,Bash,shell,diff
Author: ""
Private: false
---

#全て出力

```
$ git diff
diff --git a/file b/file
index edd13ee..d0f4198 100644
--- a/file
+++ b/file
@@ -1,3 +1,3 @@
 aaaa
 bbbb
-cccc
+dddd
```

#+/-の差分情報だけ欲しい
そんなオプションはなかった。ゴリ押しで実現

```
$ git diff --unified=0 --exit-code | grep -E "^[-\+]" | grep -vE "\-\-\-|\+\+\+"
-cccc
+dddd
```
