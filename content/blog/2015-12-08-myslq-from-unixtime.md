+++
date = "2015-12-09T15:40:00+09:00"
title = "MySQLでUnixTimeをTimestampに変換しようとしてつらかった話"
draft = false

+++

最近遭遇して大変だった話。
<!-- more -->
### 前提
Aから受け取ったデータをこちら側のBというDBに保存する。  
その過程で日時のデータはUnixTimeに変換して保存している。

だいたいこんなイメージ

|id|date|
|:------|:------|
|1|-100000|
|2|1480000|
|3|0|


### 問題点
問題はこれを外部に出力するときにUnixtimeを日付データに変換しないといけないということ…

MySQLだと [`from_unixtime`](https://dev.mysql.com/doc/refman/5.6/ja/date-and-time-functions.html#function_from-unixtime) を利用することになるのだが、この関数は0未満のデータは変換出来ない仕様になっている。

プログラムで中身を出力してEachやForeachでループさせてDateTimeモジュール的なもので書き換えてあげれば問題は無いのだけど実際はレコード数が数百万単位になるので時間がかかって仕方がない…

ということで以下の観点からSQLを模索してみた。

- UnixTimeは0以下と0以上が混在する
- 両方変換する
- 書式は `YYYY-MM-DD HH:mm:ss` とする
- 必ずSQLで処理する

### 書けたSQL
こんな感じで。

```sql
select
(case
when [column] < 0 then
 DATE_FORMAT(DATE_SUB(from_unixtime(0), INTERVAL (
  DATEDIFF(
   from_unixtime(
    ABS(
     cast(0 - cast([column] as signed) AS SIGNED)
    )
   ),
   from_unixtime(0)
  )
 ) day), '%Y-%m-%d %T')
when [column] >= 0 then
 from_unixtime(cast([column] as signed))
else null end) as [column]
from [table];
```

詳細の説明を。

- `case when [条件式] then else [return] end`
	- SQLでのIF文
- `ABS` 
	- 絶対値を取得するメソッド：UNSIGNEDみたいな符号付きのデータだと異常値が出るのでSIGNEDにCastしている
- `from_unixtime()`
	- UnixtimeからDatetimeへの変換を行う。0未満はNullを返す
- `datediff(date1, date2)`
	- date1とdate2との差分を求めるdate1の方が大きいと正数になる
- `date_sub(date1, INTERVAL [n] day)`
	- date1からn日分引く。dayだけでなく年、週、時間など様々な単位で取れる
- `date_format(date, '[format-text]')`
	- dateを[format-text]の書式へ変換する

これで0以上の時と0未満の時で異なる条件のSQLを記述することが出来た。
ベンチマークはまだだがCast周りを切り詰めて行けば処理速度も少しは上がると思う。

### 学び
- IF文はSQLでも書ける
- Unixtimeの変換は一通りは出来る
- 日付のデータはUnixTimeよりDate型で残しておきたいなぁ…
	- 仕様なら仕方ないけど対応がつらい…

	
### 追記（2015-12-09）

Caseを追加しなくても日付の変換は出来るということを教えてもらった
こんなSQLで日付の変換は正数でも負数でも両方変換出来る。

```sql
select
 date_format(
	 date_add(from_unixtime(0), interval unixtime second),
 	'%Y-%m-%d %T'
 )
from table;
```
ただし対象カラムにNullとかが入る場合はWarningされちゃうのでその辺どうにかしたいなぁとおもってる。  
[@QooQulU](https://twitter.com/QooQulU) 君、ありがとー