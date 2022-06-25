---
title: 刪除連續的上班紀錄或是連續的下班紀錄
tags:
    - Algorithm
    - JavaScript
    - 上下班打卡紀錄
    - Performance
date: 2022-06-25 23:30:00
---

## 緣由
前陣子在設計某項功能，這個功能需要將每個人在資料庫的上下班打卡紀錄，計算成工作時數。我們本來使用非常簡易的演算法，將資料預想成是一組、一組完美的上下班打卡記錄，逐一計算出每個人的工作時數：

``` typescript
type FlagType = 'CHECK_IN' | 'CHECK_OUT';
type WorkRecord = { logTime: Date; flag: FlagType };
function getWorkingHours(employeeId: string): number {
  const foundRecords: WorkRecord[] = findRecordsByEmployeeId(employeeId);
  return foundRecords.reduce(([prevRecord, workingHours], record) => {
    const { logTime, flag } = record;
    switch(prevRecord?.flag) {
      case 'CHECK_IN':
        if (flag === 'CHECK_OUT') return [record, logTime.getTime() - prevRecord.getTime()];
      case 'CHECK_OUT':
      default:
        if (flag === 'CHECK_IN') return ['CHECK_IN', 0];
        return [record, 0];
    }
  }, [null, 0] as [null | WorkRecord, number]);
}
```