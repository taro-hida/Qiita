---
ID: b85fef9a553f84a20b6c
Title: 'GoogleAppEngineへアプリケーションをデプロイしようとすると、UnicodeDecodeError: ''ascii'' codec can''t
  decode byte 0xe5 in position xx: ordinal not in range(128)が発生する'
Tags: WordPress,GoogleAppEngine,gcloud,gcp
Author: ""
Private: false
---

# 経緯
VM上に乗っているWordpress Applicationを、GCP上に移行しています。
手順はこちらを参照しました（[公式Tutorial](https://cloud.google.com/community/tutorials/run-wordpress-on-appengine-standard?hl=ja)）
wordpressのプロジェクトをデプロイ（`$ gcloud app deploy app.yaml cron.yaml`）しようとした際に問題が発生しました。

# 環境
- Windows10:
Version 1903 (OS Build 18362.657)

- powershell:

```
PS C:\Users\user_name> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.18362.628
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.18362.628
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```
- [Google Cloud SDK](https://cloud.google.com/sdk/install?hl=ja):

```
PS C:\Users\user_name> gcloud --version
Google Cloud SDK 282.0.0
beta 2019.05.17
bq 2.0.54
core 2020.02.21
gsutil 4.47
```
Google Cloud SDKはインストールしてすぐ。ホヤホヤのものです。

# 出ているのは、以下のエラー
```
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position xx: ordinal not in range(128)
```
pythonはString型などで値を保持する際、[文字コードに依存しないUnicodeという形式で値を扱っています](https://docs.python.org/ja/3/howto/unicode.html)。
そして出力の際、python2はそのUnicodeをdecodeして出力しますが、そのdecodeにおいて指定する文字コードとして、デフォルトでasciiを使用します。
python3ではutf-8で処理するのが標準になっています。
設定項目は、`sys.getdefaultencoding()`で確認できます。

```
Python 3.8.2 (tags/v3.8.2:7b3ab59, Feb 25 2020, 23:03:10) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license()" for more information.
>>> import sys
>>> sys.getdefaultencoding()
'utf-8'
```

# 原因と現象の回避策
## 解決法1 
`sys.setdefaultencoding("utf-8")`でコーデックが利用する文字コードを`ascii`から`utf-8`に変更します。
当該エラーコードで検索すると山ほど出てくる解決方法ですね。本記事は、この方法を試してだめだった人のための対応方法がメインです。
当該方法についての解決策については、以下の記事をご覧いただければと思います。
[pythonのデフォルトエンコーディングをutf-8に変更する - @puriketu99](https://qiita.com/puriketu99/items/55e04332881d7b679b00)

## 解決法2
powershellの変数に、`$CLOUDSDK_PYTHON`を指定します。

**前提**

- powershell上で`python -V`と入力すると3系のバージョンが出力されていること
  - python3系がない場合は、3系のpythonをインストールしてください。

具体的には以下コードの発行（ただの変数定義です）

```
PS C:\Users\user_name> $CLOUDSDK_PYTHON= "C:\Users\user_name\AppData\Local\Microsoft\WindowsApps\python.exe"
```
指定するパスについては、MicrosoftStore標準のpythonを指定するのが望ましいとソースコードに書いてありました。
私のpowershellはMicrosoftStore標準のpythonを利用していましたので、以下コマンドでpathを確認しています。[^1]
[^1]: Thx to @vmmhypervisor! for https://qiita.com/vmmhypervisor/items/b0fc773e49e1bbbbd5be

`where`ではなく、`where.exe`コマンドを利用しましょう。

```
PS C:\Users\user_name> where.exe python
C:\Users\user_name\AppData\Local\Microsoft\WindowsApps\python.exe
```
上記変数を設定してgcloudコマンドを実行するとあら不思議、エラーが発生することなくコマンドを実行することができるかと思います。

毎度変数を設定するのが面倒だという人は、環境変数に設定してしまうことをお勧めします。

```
PS C:\Users\user_name> powershell -Command "[System.Environment]::SetEnvironmentVariable('CLOUDSDK_PYTHON', 'C:\Users\user_name\AppData\Local\Microsoft\WindowsApps\python.exe', 'User')"
```
.bashrcみたいなもので、この変数を反映するためには一度powershellのプロセスを切って新規に起動する必要があることに注意してください。

```
PS C:\Users\user_name> $env:CLOUDSDK_PYTHON
C:\Users\user_name\AppData\Local\Microsoft\WindowsApps\python.exe
```
一度Windowを閉じて再度powershellを起動すると、環境変数が設定されていることを確認できます。

コマンドプロンプトにおいては変数をset, 環境変数をsetxで設定できますが、Microsoftが悲しそうな顔をしそうなのでMicrosoft推しのpowershellの利用を推奨します。
また、[setxの挙動には注意点がある](https://qiita.com/jeyei/items/05ce2739501832463b3b)そうで、この点からもpowershellの利用がお勧めです。[^2]
[^2]: Thx to @jeyei!!

# トラブルシュートの履歴と、原因の詳細な説明
ここからは、私のトラブルシュートの履歴と、合わせて今回のエラーについての詳細な説明をしていきます。

### pythonのバージョンを確認する
まず、python2系でよく見るエラーであったため、手元のpythonのバージョンを確認しました。
pythonのバージョンアップで治るならめっちゃ簡単で楽です。

```
PS C:\Users\user_name> python -V
Python 3.7.6 (tags/v3.7.6:43364a7ae0, Dec 19 2019, 01:54:44) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
```
3系でした。

### gcloudの実行ファイルを見てみる

```
where.exe gcloud
```

gcloudコマンドの実態は、batファイルでした。
一行目の`@echo off`をコメントアウトして、出力を出してみると,,,

```diff
- @echo off
+ rem @echo off
```
```
C:\Users\user_name>"C:\WINDOWS\system32\cmd.exe" /C ""C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\..\platform\bundledpython\python.exe" -S "C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\..\lib\gcloud.py" "
ERROR: (gcloud) Command name argument expected.
Command name argument expected.

```
上記の行でエラーが出ていました。
なるほど！実際の実行に使われているinterpreterは`C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\..\platform\bundledpython\python.exe`にあるのか！！

`C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\..\platform\bundledpython\python.exe`をエクスプローラでダブルクリックし実行してみると、

```
Python 2.7.13 (v2.7.13:a06454b1afa1, Dec 17 2016, 20:53:40) [MSC v.1500 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
```
2系であることを確認できました。

### `gcloud.py`を読んでみる
python2系のinterpreterに読み込まれている`C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\lib\gcloud.py`のファイルを以下のように編集し、現在の情報を確認してみます。

```diff
#!/usr/bin/env python
# -*- coding: utf-8 -*- #
#
# Copyright 2013 Google LLC. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

"""gcloud command line tool."""

from __future__ import absolute_import
from __future__ import division
from __future__ import unicode_literals

import os
import sys


+ print sys.getdefaultencoding()
+ sys.exit()
```

```
C:\Users\user_name>"C:\WINDOWS\system32\cmd.exe" /C ""C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\..\platform\bundledpython\python.exe" -S "C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\..\lib\gcloud.py" "
ascii
C:\Users\user_name>"C:\WINDOWS\system32\cmd.exe" /C exit 0
```
ビンゴですね！
python2系のスクリプトであり、文字コードの設定がasciiであることを確認できました。

では、この設定ファイルの冒頭で、`sys.setdefaultencoding("utf-8")`を呼び出させます。

```diff
#!/usr/bin/env python
# -*- coding: utf-8 -*- #
#
# Copyright 2013 Google LLC. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

"""gcloud command line tool."""

from __future__ import absolute_import
from __future__ import division
from __future__ import unicode_literals

import os
import sys


- print sys.getdefaultencoding()
- sys.exit()
+ sys.setdefaultencoding("utf-8")
```

このコードの変更によって、当該エラーは解消されたことを確認しました。

## もう一つの解決策
上記の通り、Google Cloud SDKにおいては、pythonのインタプリタについて`C:\Users\user_name\AppData\Local\Google\Cloud SDK\google-cloud-sdk\platform\bundledpython\python.exe`という、sdkにbundleした2系のpythonを利用しています。
これを利用せず、3系のpythonを利用するようにgcloud.cmdを編集すればいいのではないかと考えました。
正直、システムファイルである`setup.py`を手で書き換えるのは非常に気持ち悪いです。

### 再度`gcloud.cmd`を読む

`gcloud.cmd`を読むと、以下のように処理されていました。

```
IF "!CLOUDSDK_PYTHON!"=="" (
  SET BUNDLED_PYTHON=!CLOUDSDK_ROOT_DIR!\platform\bundledpython\python.exe
  IF EXIST "!BUNDLED_PYTHON!" (
    SET CLOUDSDK_PYTHON=!BUNDLED_PYTHON!
  )
)
```
`$CLOUDSDK_PYTHON`という変数が設定されていない場合は、`BUNDLED_PYTHON`のpathが適用されるという処理になっています。なるほど。

というわけで、[解決法2](#解決法2)で説明したとおりの処理を行うと、gcloudコマンドが通るようになるんですね。

```
PS C:\Users\user_name\Documents\projects\wordpress> $CLOUDSDK_PYTHON = "C:\Users\user_name\AppData\Local\Microsoft\WindowsApps\python.exe"
PS C:\Users\user_name\Documents\projects\wordpress> gcloud app deploy app.yaml cron.yaml
```
正常にデプロイすることができました。

# ところでこれ、sdkのバグでは？
たぶん、bundleされたpythonの`site-packages`に`sys.setdefaultencoding("utf-8")`って記載するだけで、当該のエラーは出なくなります。
開発者たちはおそらく英語で開発してらっしゃいますし、これは単純なバグなのではないでしょうか。bundleしているpythonでバグるのが正で、引数を渡さないと正しく動かないのだよというのであれば、そもそもbundleしないで、「Pythonのインタプリタを設定してね」とユーザに返してあげるのがいいと思います。

## googleに、issueを出してみる
Githubで公開のオープンソースとして開発されているのだろうと思って探してみましたが、見つかりませんでした。
こういうGCPなどのGoogle製品のバグ報告、要望のフィードバックについては、issue Trackerというのが用いられています。

というわけで、私もissueを立ててみました！！！（\ﾃﾞﾃﾞｰﾝ/）
https://issuetracker.google.com/issues/150633542

本当にこれがバグであって修正されるならば、いちいち環境変数を設定したりしなくてよくなるはずです。

### 追記
既知のバグだったようで、重複したissueということで`dupulicate`のStatusがつけられました。
https://issuetracker.google.com/issues/145858904

# 最後に
ここまでお読みいただきまして、ありがとうございました。
もし間違ってるところあれば、pull requestかコメントを頂けると非常に助かります。

# お世話になった記事
https://proengineer.internous.co.jp/content/columnfeature/5205
https://qiita.com/vmmhypervisor/items/b0fc773e49e1bbbbd5be
https://qiita.com/hirocueki2/items/1789998804c4e7ec1ef3
https://qiita.com/Takumi_Kaibara/items/073f17cc6e5112e4cdf7

ps. もしトラブルシュートの役に立った方がいれば泣いて喜ぶのでコメントください。
