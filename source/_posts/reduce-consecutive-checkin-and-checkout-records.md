---
title: 計算員工的工作時數
tags:
  - Algorithm
  - JavaScript
  - TypeScript
  - 上下班打卡紀錄
  - Performance
  - SQL
  - Window function
date: 2023-01-01 01:12:00
---


## 緣由
前陣子在設計某項功能，這個功能需要將每個人在資料庫的上下班打卡紀錄，計算成工作時數。我們本來使用非常簡易的演算法，將資料預想成是一組、一組完美的上下班打卡記錄，逐一計算出每個人的工作時數：

假設資料表的資料如下：

| ROW_NUMBER | EMP_NO | FLAG      | LOG_TIME                 |
| ---------- | ------ | --------- | ------------------------ |
| 1          | a00001 | CHECK_IN  | 2022-06-01T07:59:02.000Z |
| 2          | a00002 | CHECK_IN  | 2022-06-01T07:59:08.000Z |
| 3          | a00001 | CHECK_OUT | 2022-06-01T17:02:32.000Z |
| 4          | a00002 | CHECK_OUT | 2022-06-01T17:02:42.000Z |
| 5          | a00002 | CHECK_IN  | 2022-06-02T07:49:08.000Z |
| 6          | a00001 | CHECK_IN  | 2022-06-02T07:58:42.000Z |
| 7          | a00002 | CHECK_OUT | 2022-06-02T17:01:02.000Z |
| 8          | a00001 | CHECK_OUT | 2022-06-02T17:04:22.000Z |

``` typescript
type FlagType = 'CHECK_IN' | 'CHECK_OUT';
type WorkRecord = { EMP_NO: string; LOG_TIME: Date; FLAG: FlagType };
function getWorkingHours(employeeId: string): number {
  const foundRecords: WorkRecord[] = findRecordsByEmployeeId(employeeId);
  return foundRecords.reduce(([prevRecord, workingHours], record) => {
    const { LOG_TIME: CURRENT_LOG_TIME, FLAG: CURRENT_FLAG } = record;
    const { LOG_TIME: PREV_LOG_TIME, FLAG: PREV_FLAG } = prevRecord;

    // 若前一筆是上班紀錄且當前這筆是下班紀錄，則：
    // 1) 把當前的 record 作為下一筆的 prevRecord
    // 2) 疊加 workingHours 並回傳
    if (PREV_FLAG === 'CHECK_IN' && CURRENT_FLAG === 'CHECK_OUT') {
      const accHours = (LOG_TIME.getTime() - PREV_LOG_TIME.getTime()) / 1000 / 3600;
      return [record, workingHours + accHours];
    }

    // 否則：
    // 1) 把當前的 record 作為下一筆的 prevRecord
    // 2) 直接回傳 workingHours
    return [record, workingHours];
  }, [null, 0] as [null | WorkRecord, number]);
}
```

上面這段演算法很單純，只會將 `CHECK_IN` -> `CHECK_OUT` 的差值加計到上下班時數的累計時數。

## 現實
但是，事實並沒有這麼理想，因為有些員工可能會因為怕漏刷而連續刷多次上班或下班打卡記錄。這樣一來，上下班時數很有可能被計算錯誤。舉例來說，員工 `a00001` 在 `2022-06-20` 的上下班紀錄如下：

| ROW_NUMBER | EMP_NO | FLAG      | LOG_TIME                 |
| ---------- | ------ | --------- | ------------------------ |
| 1          | a00001 | CHECK_IN  | 2022-06-20T07:59:02.000Z |
| 2          | a00001 | CHECK_IN  | 2022-06-20T07:59:04.000Z |
| 3          | a00001 | CHECK_IN  | 2022-06-20T07:59:05.000Z |
| 4          | a00001 | CHECK_OUT | 2022-06-20T17:02:20.000Z |
| 5          | a00001 | CHECK_OUT | 2022-06-20T17:02:23.000Z |

如果是用上面的演算法來計算員工 `a00001` 在 `2022-06-20` 的工作時數， 第 3 筆會被認為上班時間 ，而第 4 筆則會被認為是下班時間；但是，以常理來看，上下班時間分別應該是第 1 筆和第 5 筆。

## 思路
回顧一開始的演算法，僅著重在當前紀錄與前一筆的關係。實在摸不著頭緒的時候，我把時間軸、可能的上下班紀錄畫出來。在幾番思考之後，我歸納出了一個對應表，可以將原始的上下班紀錄轉換為成對的上下班紀錄。

| CASE NO. | T1(PREV)  | T2(CURRENT) | T3(NEXT)  | WORK_START | WORK_END |
| -------- | --------- | ----------- | --------- | ---------- | -------- |
| 1        | CHECK_IN  | CHECK_IN    | CHECK_IN  |            |          |
| 2        | CHECK_IN  | CHECK_IN    | CHECK_OUT |            |          |
| 3        | CHECK_IN  | CHECK_OUT   | CHECK_IN  |            | T2       |
| 4        | CHECK_IN  | CHECK_OUT   | CHECK_OUT |            |          |
| 5        | CHECK_OUT | CHECK_IN    | CHECK_IN  | T2         |          |
| 6        | CHECK_OUT | CHECK_IN    | CHECK_OUT | T2         |          |
| 7        | CHECK_OUT | CHECK_OUT   | CHECK_IN  |            | T2       |
| 8        | CHECK_OUT | CHECK_OUT   | CHECK_OUT |            |          |

在歸納出這張表之前，不斷地思考如何找出「能夠計入上下班的紀錄」，反覆地看著時間軸和其上面的上下班紀錄。基本上可以看出兩個規則：
1. 若當前是上班紀錄且前一筆是下班紀錄，則當前這筆上班紀錄是最後要計入時數的起算時間點
2. 若當前是下班紀錄且下一筆是上班紀錄，則當前這筆下班紀錄是最後要計入時數的結算時間點

其餘的紀錄就把它排除囉～

所以，先做一次資料梳理，就能接著使用原本的工作時數演算法了。

``` typescript
type FlagType = 'CHECK_IN' | 'CHECK_OUT';
type WorkRecord = { EMP_NO: string; LOG_TIME: Date; FLAG: FlagType };

function getWorkingHours(employeeId: string): number {
  const foundRecords: WorkRecord[] = findRecordsByEmployeeId(employeeId);
  return foundRecords
    .filter((record, index, allRecords) => {
      // 因為接下來會看前一筆和後一筆，但是第一筆沒有前一筆資料、最後一筆沒有下一筆資料，所以要特殊處理、直接計入
      if (index === 0 || index === allRecords.length - 1) return true;
      const { FLAG: PREV_FLAG } = allRecords[index - 1];
      const { FLAG: NEXT_FLAG } = allRecords[index + 1];
      const { FLAG: CURRENT_FLAG } = record;
      if (CURRENT_FLAG === 'CHECK_IN' && PREV_FLAG === 'CHECK_OUT') return true;
      if (CURRENT_FLAG === 'CHECK_OUT' && NEXT_FLAG === 'CHECK_IN') return true;
      return false;
    })
    .reduce(([prevRecord, workingHours], record) => {
      const { LOG_TIME: CURRENT_LOG_TIME, FLAG: CURRENT_FLAG } = record;
      const { LOG_TIME: PREV_LOG_TIME, FLAG: PREV_FLAG } = prevRecord;

      // 若前一筆是上班紀錄且當前這筆是下班紀錄，則：
      // 1) 把當前的 record 作為下一筆的 prevRecord
      // 2) 疊加 workingHours 並回傳
      if (PREV_FLAG === 'CHECK_IN' && CURRENT_FLAG === 'CHECK_OUT') {
        const accHours = (LOG_TIME.getTime() - PREV_LOG_TIME.getTime()) / 1000 / 3600;
        return [record, workingHours + accHours];
      }

      // 否則：
      // 1) 把當前的 record 作為下一筆的 prevRecord
      // 2) 直接回傳 workingHours
      return [record, workingHours];
    }, [null, 0] as [null | WorkRecord, number]);
}
```

上面的演算法時間複雜度為 `O(n)` ，效率還蠻不錯的。在沒有列出排列組合表分析之前，本想打算用暴力的方式用兩層迴圈來篩選資料，但是一次要跑幾十萬筆資料的情況下，會讓 CPU 爆炸，於是就放棄了。

## WINDOW FUNCTION
最近剛好學到 WINDOW FUNCTION ，馬上就聯想到這個例子可以用這個方法解，效率應該不會差到哪去，且可以把 accumulation 也一起交給 SQL 來做，這樣省下了不少功夫...

``` SQL
/* PostgreSQL */
/* 用 CTE 把 query 一個一個串起來、比較好閱讀 */
WITH
  /* 先把每一筆紀錄的前後紀錄找出來，放在同一筆 row */
	working_records AS (
		SELECT emp_no,
			LAG(flag, 1) OVER (PARTITION BY emp_no ORDER BY log_time ASC) AS prev_record_flag,
			LAG(log_time, 1) OVER (PARTITION BY emp_no ORDER BY log_time ASC) AS prev_record_time,
			flag AS current_flag,
			log_time AS current_record_time,
			LEAD(flag, 1) OVER (PARTITION BY emp_no ORDER BY log_time ASC) AS next_record_flag,
			LEAD(log_time, 1) OVER (PARTITION BY emp_no ORDER BY log_time ASC) AS next_record_time
		FROM test_anson_employee_working_records
	),

  /* 套用剛剛提到的規則篩選出要留下的紀錄 */
  filtered_working_records AS (
    SELECT emp_no, current_flag, current_record_time
    FROM working_records
    WHERE 
      (current_flag = 'CHECK_IN' AND prev_record_flag is null) or
      (current_flag = 'CHECK_IN' AND prev_record_flag = 'CHECK_OUT') or
      (current_flag = 'CHECK_OUT' AND next_record_flag = 'CHECK_IN') or
      (current_flag = 'CHECK_OUT' AND next_record_flag is null)
  ),

  working_hours AS (
    /* 找出每一筆上班紀錄，然後再用一次 `LEAD` 找出下一筆下班紀錄(因為已經篩選過，所以資料都是上下班按照順序排序)算出當次的上班時間 */
    /* 待修正：沒辦法正確算出 diff ，可能是 PARTITION 的問題？ */
    SELECT emp_no, LEAD(current_record_time, 1) OVER (PARTITION BY emp_no ORDER BY current_record_time) - current_record_time
    FROM filtered_working_records
    WHERE current_flag = 'CHECK_IN'
  )

/* 加總... */
```
