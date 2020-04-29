---
ID: bfbd06637dc99e1f2042
Title: PostgreSQLでCOPY table TO file WITH CSVするときに、FORCE QUOTE *とするとエラーが出る
Tags: Bash,SQL,PostgreSQL,CSV
Author: ""
Private: false
---

下記のようなエラーが出ます。

```sql
COPY your_table_name TO 'tukareta.csv' WITH CSV DELIMITER ',' FORCE QUOTE * NULL AS '' HEADER;
ERROR:  syntax error at or near "*"
```

これはpostgresのバージョンが8系の場合、FORCE QUOTEの引数に`*`が定義されていないことにより起こる問題です。ドキュメントの一部を抜粋すると以下のようになります。

```
# バージョン9.2の場合
# https://www.postgresql.jp/document/9.2/html/sql-copy.html

 FORCE_QUOTE { ( column_name [, ...] ) | * }

# バージョン8.4の場合
# https://www.postgresql.jp/document/8.4/html/sql-copy.html

[ FORCE QUOTE column [, ...] ]
```

はい。バージョン8.4側の定義に`*`がありませんね。
原因がバージョンによる仕様ということが分かったところで、解決策行きましょう。

```tukareta.sh
COLUMN_NAME=`psql -d your_db_name -U your_username -tc "SELECT column_name FROM information_schema.columns WHERE table_name = 'your_table_name' ORDER BY ordinal_position;"`
CONMA_COLUMN=`echo ${COLUMN_NAME}|sed 's/\s/,/g'`
SQL="COPY your_table_name TO 'tukareta.csv' WITH CSV DELIMITER ',' FORCE QUOTE "${CONMA_COLUMN}" NULL AS '' HEADER;"
echo "${SQL}" | psql \
  -d your_dbname \
  -U your_username \

```

####シェルスクリプトドーン！！！

すみません、SQLの形で作れないどころか、複数行のシェルスクリプトになりました。でも、これで「FORCE QUOTE *」の際と同じ挙動をしてくれることを確認しています。

何をやっているかというと、*の部分に、カラム名を`,`区切りで入力しています。

```
[ FORCE QUOTE column [, ...] ]
```
繰り返しますが、8系の定義は上記ですので、

```
FORCE QUOTE id,name,value, ...
```
こんな感じでカラム名を全部`*`のところに列挙すればいいわけですね（カンマ区切り）。
サンプルですが、発行するSQL文は以下のようになっています。

```bash
# 上記スクリプトに追記
+ echo "${SQL}"

# 出力
COPY testtable TO 'tukareta.csv' WITH CSV DELIMITER ',' FORCE QUOTE id,name,value,timestamp NULL AS '' HEADER;
```
ちなみに、postgres9系の場合は以下のようなSQL一行で同じことができますね。

```sql
COPY your_table_name TO 'tukareta.csv' WITH CSV DELIMITER ',' FORCE QUOTE * NULL AS '' HEADER;
```
というわけで、バージョンを上げれる環境の人は頑張らずに、バージョンを上げてしまうことを推奨します。

ちなみに私は今回、複数のテーブルから情報を引っ張る必要があったので、以下のように上記のコードを複数回ループさせました。

```
#!/bin/bash

TABLES=(table1 table2 table3 table4 table5 table6 )

for TABLE in ${TABLES[@]}
do
COLUMN_NAME=`psql -d hogedb -U hogeuser -tc "SELECT column_name FROM information_schema.columns WHERE table_name = '"$TABLE"' ORDER BY ordinal_position;"`
CONMA_COLUMN=`echo ${COLUMN_NAME}|sed 's/\s/,/g'`
SQL="COPY "${TABLE}" TO '/tmp/"${TABLE}".csv' WITH CSV DELIMITER ',' FORCE QUOTE "${CONMA_COLUMN}" NULL AS '' HEADER;"

echo $CONMA_COLUMN

echo "${SQL}" | psql \
  -d hogedb \
  -U hogeuser \

done

```
もしそのまま使えたなら使っていただければと思います。お役に立てていたらうれしいです。

#余談

おそらく、このエラーについて調べていて一番最初に開くのは以下のブログではないでしょうか？
[PostgreSQL の CSV エクスポートでエラーが出る - 電気ウナギ的○○](http://blog.netandfield.com/shar/2018/05/postgresql-csv.html)


>バージョンは PostgreSQL 8.4.20 でちょっと古いんだけど、8 の時ってワイルドカード、使えんかったっけ？
>確かに、PostgreSQL 9.0.4 だったらエラーにならないなあ。

って書かれてますが、この予想は当たっていたわけですね。
おかげさまで僕も正解にたどり着けました。

ちなみに上記のコードですが、もう少しきれいに書ける気がするんですよね。
もしリファクタリング等していただけた方がいらっしゃいましたら記事のマージリクエストください。喜んでマージします。マジで。

