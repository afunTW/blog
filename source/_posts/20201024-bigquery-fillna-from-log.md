---
title: BigQuery - fill data from log
date: 2020-10-24 00:00:00
updated: 2020-11-20 00:00:45
categories: Data Engineering
tags: [bigquery, sql]
---
> SQL 寫的好，要飯要到老，今天就來分享一下要飯的幾個小技巧

<!-- more -->

資料科學領域中，寫 SQL 拿資料是非常基礎也非常常用的需求，各式各樣的 Database 都有其效能與特殊寫法，而最近因為雲端化 data warehouse 的需求比較頻繁的寫 bigquery，這邊也順便紀錄一下幾個情境

雲端化的資料管理中，data lake 是所有資料的最上游，與傳統資料庫的設計中遵循著 ER or EER model 並不完全相同，同時包含著各種非結構化的資料型式

- 資料庫表格：被設計過的表格型式，通常會有 Primary Key (PK) 或是 Foreign Key (FK) 供參考操作
- Event Log：根據 timestamp 紀錄事件發生的先後順序
- ...

建立 warehouse 的過程中經常需要將資料庫表格與 event log 兩種資料型式做合併處理，除了要釐清商業邏輯以外，也時常要了解兩種機制設計上的衝突與概念

- Event Log 每次打上來的時間間隔不固定的原因？
- Event Log 的 exception 機制處理？
- 事件觸發會同時紀錄在表格跟 log？

除了需要大量的溝通與理解以外，還要透過 SQL 做 data cleaning，這邊以 bigquery 為例紀錄一下我常遇到的情境

# 從 Log 中取得上一筆最近的資料做補值

> 核心概念：將資料表與 Log 做 union 後根據 timestamp 排序，再透過 window function 取值

經常發生在當表格的資料是某個時間點的 snapshot 而不是過去一段時間的資料做 aggregation 時，某些欄位的值不會頻繁更新但卻偶爾會有漏資料的情況

假設現在需要從以下兩張表格處理，取得換電池紀錄中每一顆電池的全充容量

- switch_battery 換電紀錄 (資料庫表格)
    - service_id (PK)：換電紀錄的序號 (PK)
    - battery_id：被更換的電池序號
    - timestamp：換電紀錄發生的時間點
- battery_bms_log 電池管理系統 (Event Log)
    - battery_id：電池序號
    - full_capacity：電池充滿電的容量（全充容量）
    - timestamp：電池管理系統內紀錄電池全充容量的時間點

電池會因為老化而導致全充容量的值遞減，但因為老化不會在短時間內發生，所以可以從 log 合併最接近時間點的全充容量

**要取得上一筆最近時間的資料，也就是要將兩種格式整理為同一張表格，並依照各個電池事件的時間排序**

```sql
WITH
-- 因為沒有 key 可以參考所以無法使用 JOIN 的方式合併表格
-- 常用作法是建立假的欄位再透過 UNION ALL 的方式合併
-- 為了更明確辨認資料來源，可以自定義欄位當作 anchor
UNION_RECORD_BMS AS (
    SELECT
        service_id,
        battery_id,
        NULL AS full_capacity, -- 建立假的欄位
        timestamp,
        'switch' AS data_anchor
    FROM `myproject.mydataset.switch_battery`
    UNION ALL
    SELECT
        NULL AS service_id
        battery_id,
        full_capacity, -- 建立假的欄位
        timestamp,
        'log' AS data_anchor
    FROM `myproject.mydataset.battery_bms_log`
),
-- 透過 window function 取得過去最近一筆時間點的資料
GET_PREV_CAPACITY AS (
    SELECT
        service_id,
        battery_id,
        timestamp,
        LEAD(full_capacity) OVER w AS full_capacity,
        LEAD(timestamp) OVER w AS bms_timestamp,
        data_anchor
    FROM UNION_RECORD_BMS
    WINDOW w AS (
        PARTITION BY battery_id
        ORDER BY timestamp DESC
    )
)
-- 整理欄位並根據 anchor 將 log 的資訊排除
SELECT * EXCEPT (data_anchor)
FROM GET_PREV_CAPACITY
WHERE data_anchor = 'switch'
```

# 確保補值的資料來源是 Log

> 核心概念：透過 anchor 確保補值的資料來源

延續範例，我們可以進一步考慮現實世界的系統是不穩定的環境

在兩次換電紀錄的過程中，如果系統發生異常，有可能會導致整段 Log 遺失，進而讓我們在 `LEAD` 的地方發生邏輯錯誤

**發生的時間順序 (Acending)**

1. `2020-06-01T12:31:00` log
2. `2020-06-01T12:30:00` switch
3. `2020-06-01T12:00:00` switch
4. `2020-06-01T11:59:00` log
5. `2020-06-01T11:58:00` log
6. ...

假設 `12:00:00`~ `12:30:00` 中間的 log 都因為系統異常而遺失，(2) 透過 window function 取得的資料就都會來自於 (3) 但同樣屬於 switch，這種狀況下我們可以根據 anchor 判斷資料來源

```sql
...
GET_PREV_CAPACITY AS (
    SELECT
        service_id,
        battery_id,
        timestamp,
        LEAD(full_capacity) OVER w AS prev_full_capacity,
        LEAD(timestamp) OVER w AS bms_timestamp,
        data_anchor,
        full_capacity, -- 保留原本自己的資料
        LEAD(data_anchor) OVER w AS prev_data_anchor -- 保留 reference anchor
    FROM UNION_RECORD_BMS
    WINDOW w AS (
        PARTITION BY battery_id
        ORDER BY timestamp DESC
    )
)
-- 根據 windown function 的結果確保資料來源
-- 如果資料來源不是 log 則強制指定值為 NULL
SELECT
    service_id,
    battery_id,
    timestamp,
    IF(prev_data_anchor = 'log', 
       prev_full_capacity,
       NULL
    ) AS full_capacity
FROM GET_PREV_CAPACITY
WHERE data_anchor = 'switch'
```

補充：這邊的應用很彈性，假設當情境改為「資料表與 log 都有同一個欄位的資料，但是要從 log 做補值」時，可以在 `IF` statement 中使用 `COALESCE` 做補值

```sql
IF(prev_data_anchor = 'log', 
   COALESCE(full_capacity, prev_full_capacity),
   full_capacity
) AS full_capacity
```

# 從 Log 中取得上一筆最近的非空值資料做補值

> 核心概念：熟悉 window function 的完整語法

與一開始設定的情境很像，但是關注的點改為 window function 捕捉的範圍

[https://cloud.google.com/bigquery/docs/reference/standard-sql/analytic-function-concepts#syntax](https://cloud.google.com/bigquery/docs/reference/standard-sql/analytic-function-concepts#syntax)

```
analytic_function_name ( [ argument_list ] ) OVER over_clause

over_clause:
  { named_window | ( [ window_specification ] ) }

window_specification:
  [ named_window ]
  [ PARTITION BY partition_expression [, ...] ]
  [ ORDER BY expression [ { ASC | DESC }  ] [, ...] ]
  [ window_frame_clause ]

window_frame_clause:
  { rows_range } { frame_start | frame_between }

rows_range:
  { ROWS | RANGE }
```

我們經常使用的是透過 `ORDER BY` 的順序搭配 `LEAD` 或是 `LOG` 來取得最近的前一筆或是後一筆資料

這其實限制了 window 的可視範圍只有在前後一筆，如果我們需要的值在這個範圍之外就參考不到了

延續上面的範例，當我們條件改為要尋找過去第一筆非空值，就變成要修改 frame clause 與 rows_range 的部份了

```sql
...
-- ORDER BY 順序是 decending, 所以 FIRST_VALUE 會變成取得過去第一筆資訊
-- RANGE 那段則是要考慮可視範圍
GET_PREV_CAPACITY AS (
    SELECT
        service_id,
        battery_id,
        timestamp,
        FIRST_VALUE(full_capacity IGNORE NULLS) OVER w AS full_capacity,
        data_anchor,
    FROM UNION_RECORD_BMS
    WINDOW w AS (
        PARTITION BY battery_id
        ORDER BY timestamp DESC
        RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
    )
)
```
