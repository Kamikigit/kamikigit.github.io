+++
date = '2025-11-15T16:50:57+09:00'
lastmod = '2025-11-15T16:50:57+09:00'
draft = true
tags = ["BigQuery", "GA4", "GSC"]
category = "Howto"
subcategory = ""
title = 'GA4とGSCのタイムゾーンを日本時刻へ変換する'
+++

GSCとGA4のデータは公式のコネクタを通してBigQueryに保存することができますが、その日付データのタイムゾーンはどちらも日本時刻ではありません。

日本時刻への変換の仕方を記載します。

## Google Analytics4 のタイムゾーン

`event_timestamp`列の値は、**世界標準時（UTC）** が採用されています。[^1] ロンドン時刻です。
このタイムゾーンを変更することはできません。

`event_date`列は、GA4プロパティのレポート用タイムゾーンなのでユーザが変更することができます。GA4にて「設定」>「プロパティ」>「プロパティの詳細」からタイムゾーンを設定できます。ここは日本時刻に設定推奨です。


### 日本時刻への変換方法

```sql
SELECT
    DATETIME(TIMESTAMP_MICROS(event_timestamp), 'Asia/Tokyo') AS date_time
FROM `your_project.your_dataset.your_table`
WHERE _TABLE_SUFFIX = '20250101'
```

このコードの2行目のように、DATETIME関数を使用して`event_timestamp`列を日本時刻に変換することができます。

ちなみに`_TABLE_SUFFIX`は、GA4プロパティで指定したタイムゾーンに基づいています。[^2] なので日本のタイムゾーンに設定していれば問題ないです。




## Google Search Consoleのタイムゾーン

`data_date`列の値は、**太平洋時間（PT）** が採用されています。[^3]
このタイムゾーンを変更することはできません。

### PTとは？

アメリカ西海岸（ロサンゼルスとか）で採用されている時間のようです。

厄介な点としては、夏時間と冬時間があるので季節によって1時間ずれます。

- 冬（11月～3月）：UTC-8時間（PST）
- 夏（4月～10月）：UTC-7時間（PDT）

夏であれば、PTに17時間を足すことで日本時刻になります。冬は16時間です。


### 日本時刻への変換方法

残念なのですが、GSCの`data_date`フィールドに時刻を含まないので、正確に日本時刻へ変換することは不可能です。ざっくりと計算するしかありません。

 
例えば、日本時刻での**1月1日 0時〜24時**は、GSCでは**12月31日 8時　〜 1月1日 8時**で記録されます。

なので日本時刻での1月1日のGSCデータを知りたい場合、`data_date`フィールドにて**66%の確率で12月31日、33%の確率で1月1日**のデータとして記録されていることになります。

ですが大抵のwebサイトは深夜や朝ではなく、夕方〜夜にかけてのユーザが多いと思います。そのため確率は半分くらいになるのではないでしょうか。

<br>

難しいですね。
とりあえず、**日本時刻のデータを取りたいときは前日も検索に含めて考える** というのが良さそうです。

---

[^1]: "The time (in microseconds, UTC) when the event was received by Google Analytics." [Google Analytics Help - BigQuery Export schema](https://support.google.com/analytics/answer/7029846?hl=en)
[^2]: "Updates to the tables that are created as part of BigQuery Export are governed by the time zone of the Analytics property from which data is being exported." [Google Analytics Help - BigQuery Export](https://support.google.com/analytics/answer/9358801)
[^3]: "data_date: The day on which the data in this row was generated (Pacific Time)." [Google Search Console Help - Search Console BigQuery Export schema](https://support.google.com/webmasters/answer/12917991)
