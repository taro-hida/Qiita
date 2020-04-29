---
ID: 5ec1ba1a27f92769a649
Title: '[python]何かに囲まれた間のテキストを取得したい'
Tags: Python
Author: ""
Private: false
---

見つからなかったので作成して公開。

#Thanks！！
2018/09/15 @uasiさんから編集リクエストをいただき、マージしました！ありがとう！！！

```extract_text.py
- match_obj = re.search(begin + '(.*)' + end), text)
+ match_obj = re.search(re.escape(begin) + '(.*)' + re.escape(end), text)
```
#コード

```extract_text.py
def extract_text(begin, end, text):
	match_obj = re.search(re.escape(begin) + '(.*)' + re.escape(end), text)
	pretty_text = match_obj.groups()[0]
	return pretty_text
```
#注意点
re.searchで取得してるので、最初にマッチしたもののみ取得します。


#メモ
htmlタグのタグ内のテキストコンテンツを取得する時とかに使えると思います（そういう用途で作った）。

※今の使い方なら大丈夫だけど、.*というマッチではダメな場合も出てくるかも？？？
-> https://blog.mah-lab.com/2014/06/07/regular-expression-lazy-quantifiers/
