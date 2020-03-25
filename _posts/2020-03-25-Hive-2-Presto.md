---
toc: true
layout: post
description: Hive<>Presto function mapping
categories: [markdown][Hive][SQL][Presto][DataScience]
title: Hive2Presto
---

# Hive<>Presto function mapping

- [:cyclone: Array functions](#-cyclone--array-functions)
    + [Explode Array](#explode-array) | [Array Size](#array-size) | [Array to String](#array-to-string) | [Array Contains](#array-contains)
- [:card_index: Struct functions](#-card-index--struct-functions)
    + [cDim index](#cdim-index)
- [:date: Date functions](#-date--date-functions)
    + [UTC to User timezone](#utc-to-user-timezone) | [Day of week](#day-of-week) | [Date Diff](#date-diff) | [Date Add](#date-add) | [Date Trunc](#date-trunc) | [Hour Add](#hour-add) | [Date Filters (single day)](#date-filters--single-day-) | [Date Filters (range)](#date-filters--range-) | [Get Week begin date](#get-week-begin-date)
- [:alarm_clock: Timestamp functions](#-alarm-clock--timestamp-functions)
    + [Timestamp to date](#timestamp-to-date) | [EPOCH/Unixtime to Timestamp](#epoch-unixtime-to-timestamp) | [Timestamp to EPOCH/Unixtime](#timestamp-to-epoch-unixtime) | [Get Hour](#get-hour) | [Timestamp Filters (as date range)](#timestamp-filters--as-date-range-) | [Timestamp Filters (as time range)](#timestamp-filters--as-time-range-) | [String to Date](#string-to-date)
- [:capital_abcd: String functions](#-capital-abcd--string-functions)
    + [Concat](#concat)
- [:hash: Hash functions](#-hash--hash-functions)
    + [Hash functions](#hash-functions) | [Hash mod functions](#hash-mod-functions)
- [:link: Aggregate functions](#-link--aggregate-functions)
- [:musical_keyboard: Grouping Sets, Cube, Rollup functions](#-musical-keyboard--grouping-sets--cube--rollup-functions)
    + [Rollup](#rollup) | [Cube](#cube) | [Grouping Sets](#grouping-sets)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## :cyclone: Array functions
#### Explode Array
```sql
--Example #1: Hits
--inner join
Presto: CROSS JOIN UNNEST(hits)  as hits_table(hit)
Hive: LATERAL VIEW EXPLODE(hits) hits_table as hit
--outer join
Hive outer join: LATERAL VIEW OUTER EXPLODE(hits) hits_table as hit
Presto: --No outer join equivalent in Presto
```

```sql
--Example #2: Custom Dimensions
--inner join
Presto: CROSS JOIN UNNEST(customdimensions)  as cd_table(cd)
Hive: LATERAL VIEW EXPLODE(customdimensions) hits_table as cd
--outer join
Hive outer join: LATERAL VIEW OUTER EXPLODE(customdimensions) hits_table as cd
Presto: --No outer join equivalent in Presto
```
#### Array Size
```sql
Presto:cardinality(array)
Hive:size(array)
```
#### Array to String
```sql
Presto: array_join(array, ‘,’)
Hive: concat_ws(‘,’,array)
```
#### Array Contains
```sql
Presto: contains(array, element)
Hive: array_contains(array,element)
```
## :card_index: Struct functions
#### cDim index
```sql
Presto: exploded_cd.index=“4"
Hive: exploded_cd.index=4
```
## :date: Date functions
#### UTC to User timezone 
```sql
Presto: at_timezone(cm.event_timestamp,users.timezone)
Hive:from_utc_timestamp(cm.event_timestamp,users.timezone)
```
#### Day of week
```sql
Presto: day_of_week(mydate)
Hive: from_unixtime(unix_timestamp(timestamp1),'EEEE')
```

#### Date Diff 
```sql
Presto: date_diff('day', timestamp1, timestamp2) --supports week (starts on ??), month, quarter, year
Hive: datediff(string enddate, string startdate)
Vertica: datediff('day', timestamp1, timestamp2) --supports week (starts on Sunday), month, quarter, year
```
#### Date Add 
```sql
Presto: date_add(unit, value, timestamp) 
Hive: date_add(date/timestamp/string startdate, tinyint/smallint/int days)

--Example:
Presto: date_add(‘day’,1,cast(created as date)) 
Hive: date_add(created ,1)
Vertica: created+1  (or)  ‘2016-01-01’::date + 1
```

#### Date Trunc 
```sql
Presto: date_trunc('month', cast('2015-03-17' as date))    -- Returns 2015-03-01. Supports timestamp input. Supports day, week, month, quarter, year, hour, minute, second, millisecond
Hive: trunc('2015-03-17', 'MM')    -- Returns 2015-03-01. Supports date input only, Supports MM, YY
Vertica: date_trunc('WEEK',date '2015-03-17') --Return Week starts on Monday. Supports HOUR , MONTH, YEAR etc.,
```
#### Hour Add 
```sql
Presto: date_add('hour',7,cast(created as timestamp)) 
Hive: from_unixtime(unix_timestamp(cast(created as timestamp)) + 25200)
```

#### Date Filters (single day) 
```sql
--Case #1: When date is stored in a date column & is non partition column:
Presto: where report date=date '2017-06-05'
Hive: where report_date=date '2017-06-05'  
```
```
lang=sql
--Case #2: When date is stored in a string column (partition date columns are internally treated as string)
Presto: where cast(report_date as date)=date '2017-06-05'
Hive: where datestr='2017-06-05’ 
```

#### Date Filters (range) 
```sql
--Case #1: When date is stored in a date column & is non partition column:
Presto: where report_date between date '2017-06-05'  and date '2017-06-06'
Hive: where report_date between '2017-06-05’  and  '2017-06-06' --Note: Presto syntax works too
```
```
lang=sql
--Case #2: When date is stored in a string column (partition date columns are internally treated as string)
Presto: where cast(report_date as date) between cast('2017-06-05' as date)  and cast('2017-06-06' as date) 
Hive: datestr between '2017-06-05’  and '2017-06-06’ 
```

#### Get Week begin date 
```sql
Presto: date_trunc('week',cast(datestr as date)) week_beg_dt
Hive: date_sub(next_day(datestr,'MO'),7) week_beg_dt
```

## :alarm_clock: Timestamp functions
#### Timestamp to date 
```sql
Presto: cast(timestamp as date)
Hive: to_date(timestamp/string)  --Eg: to_date('2016-07-01 00:12:00'), to_date(current_timestamp)
Vertica: to_date ( expression , pattern ) --Eg: to_date('2016-07-01 00:12:00', 'YYYY-MM-DD HH24:MI:SS')
```

#### EPOCH/Unixtime to Timestamp 
```sql
Presto: from_unixtime(unixtime) 
Hive: from_unixtime(bigint unixtime[, string format])
```

#### Timestamp to EPOCH/Unixtime 
```sql
Presto:to_unixtime(timestamp) 
Hive: unix_timestamp(string/date/timestamp) --Eg: unix_timestamp('2016-07-01 00:12:00'), unix_timestamp(current_date)
```
#### Get Hour 
```sql
Presto: hour(timestamp)
Hive: hour(timestamp)
```
#### Timestamp Filters (as date range) 
```sql
Presto: where cast(rider_signup_succ_timestamp as date) between date '2017-06-01' and date '2017-06-01'
Hive: where to_date(rider_signup_succ_timestamp) between '2017-06-01' and '2017-06-02'
```

#### Timestamp Filters (as time range) 
```sql
Presto: cast(date_parse(created,'%Y-%m-%d %H:%i:%s') as date)
Hive: from_unixtime(unix_timestamp(created,'yyyy-MM-dd') ,'yyyy-MM-dd')
```

#### String to Date 
```sql
--"created" is formatted as String, and it looks like "2017-10-13 18:48:12"

Presto:  cast(date_parse(created,'%Y-%m-%d %H:%i:%s') as date)
Hive: from_unixtime(unix_timestamp(created,'yyyy-MM-dd') ,'yyyy-MM-dd') 
```

## :capital_abcd: String functions 
#### Concat 
```sql
Presto: concat(strcol1,strcol2,..)
Hive: concat(strcol1,strcol2,..)
```
## :hash: Hash functions 
#### Hash functions 
```sql
--MD5
Presto: md5(concat(user_uuid,'yourseed'))
Vertica: md5(concat(user_uuid,'yourseed'))

--SHA32
Presto: sha32(concat(user_uuid,'yourseed'))
Vertica: sha32(concat(user_uuid,'yourseed'))

```

#### Hash mod functions 
```sql
--MD5
Presto: 
Vertica: hex_to_integer(right(md5(concat(user_uuid,'yourseed')),15))%100 . --Creates 100 bins
```


## :link: Aggregate functions 

```sql
Presto: approx_percentile(x, percentage)
Hive: percentile(x, percentage)
Vertica: approximate_percentile(x USING PARAMETERS percentile=percentage) 
```

## :musical_keyboard: Grouping Sets, Cube, Rollup functions
#### Rollup
```sql
Presto: GROUP BY ROLLUP (col1, col2, col3)
Hive: GROUP BY col1, col2, col3 WITH ROLLUP
```

#### Cube
```sql
Presto: GROUP BY CUBE (col1, col2, col3)
Hive: GROUP BY col1, col2, col3 WITH CUBE
```

#### Grouping Sets 
```sql
Presto: GROUP BY GROUPING SETS (col1, (col1, col2), (col2,col3))
Hive: GROUP BY col1, col2, col3 WITH GROUPING SETS (col1, (col1,col2), (col2, col3))
```
