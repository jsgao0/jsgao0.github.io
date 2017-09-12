---
title: 原始型別與包覆器的差異
tags: 
    - JavaScript
    - ECMAScript
    - primitive
    - wrapper
date: 2017-09-12 10:26:02
---

## 說明
在*JavaScript*中，有五種原始型別：數值、字串、布林、`null`和`undefined`，除此`null`和`undefined`之外，其餘三種都有*原始型別包覆物件*(*primitive wrapper objects*)。 其對應的三種包覆物件有對應的內建建構式，分別為`Number()`、`String()`和`Boolean()`。

## 兩者的差異

### 最簡單的辨別方式
用包覆器建立的數值，型別都會是`object`；直接以原始型別指派給變數的話，其變數的型別則為對應的原始型別。
``` javascript
var bool = true,
    str = "example",
    num = 999;
typeof bool; // "boolean"
typeof str; // "string"
typeof num; // "number"

var boolObj = new Boolean(true),
    strObj = new String("example"),
    numObj = new Number(999);
typeof boolObj; // "object"
typeof strObj; // "object"
typeof numObj; // "object"
```

### Boolean包覆器
你覺得下面的結果會是什麼？ 提示：它用建構式，所以會是一個物件。
``` javascript
var boolObj = new Boolean(false);
console.log(boolObj);
```
答案就是`true`。 因為無論你放入任何初始化的直進去建構式，一定都會被`new`運算子回傳一個物件。 而在*js*中，物件都被視為**truthy**。 換句話說，用`new Boolean()`回傳的值，全部都是`true`，沒有例外。

如果不用`new`運算子的話，則回傳和[`truthy`](https://developer.mozilla.org/zh-TW/docs/Glossary/Truthy)和[`falsy`](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)的分類結果一樣的對應值：
``` javascript
var boolObjArray = [
  Boolean(NaN), // false
  Boolean(Number.NEGATIVE_INFINITY), // true
  Boolean(Number.POSITIVE_INFINITY), // true
  Boolean(null), // false
  Boolean(undefined), // false
  Boolean(0), // false
  Boolean(4), // true
  Boolean(''), // false
  Boolean('test'), // true
  Boolean(true), // true
  Boolean(false) // false
];
```

仔細想想，這樣的用法好像跟`!!`運算子很相似：
``` javascript
var boolObjArray = [
  Boolean(NaN), // false
  Boolean(Number.NEGATIVE_INFINITY), // true
  Boolean(Number.POSITIVE_INFINITY), // true
  Boolean(null), // false
  Boolean(undefined), // false
  Boolean(0), // false
  Boolean(4), // true
  Boolean(''), // false
  Boolean('test'), // true
  Boolean(true), // true
  Boolean(false) // false
], originalArray = [
  NaN,
  Number.NEGATIVE_INFINITY,
  Number.POSITIVE_INFINITY,
  null,
  undefined,
  0,
  4,
  '',
  'test',
  true,
  false
], isSame = boolObjArray.every(function(bool, index) {
    return bool === !!originalArray[index];
});

console.log(isSame); // true
``` 

### String包覆器
### Number包覆器