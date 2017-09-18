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
    num = 999,
    str = "example";
typeof bool; // "boolean"
typeof num; // "number"
typeof str; // "string"

var boolObj = new Boolean(true),
    numObj = new Number(999),
    strObj = new String("example");
typeof boolObj; // "object"
typeof numObj; // "object"
typeof strObj; // "object"
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

### Number包覆器
相較於`Boolean()`，`Number()`似乎就比較有用一些。 它的主要用途是作為數值判定和型別轉換，請參考 [MDN - Number#說明](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Number#說明) 。

``` javascript
Number(0) // 0
Number(123) // 123
Number(Infinity) // Infinity
Number(Number.POSITIVE_INFINITY) // Infinity
Number(Number.NEGATIVE_INFINITY) // -Infinity
Number(Number.EPSILON) // 2.220446049250313e-16
Number(Number.MAX_SAFE_INTEGER) // 9007199254740991
Number(Number.MAX_VALUE) // 1.7976931348623157e+308
Number(Number.MIN_SAFE_INTEGER) // -9007199254740991
Number(Number.MIN_VALUE) // 5e-324
Number(null) // 0
Number('') // 0
Number('123') // 123
Number(new Date('Sep 14, 2017 23:36:45')) // 1505403405000
Number(true) // 1
Number(false) // 0
Number(Boolean(true)) // 1
Number(Boolean(false)) // 0

Number(Number.NaN) // NaN
Number(NaN) // NaN
Number(undefined) // NaN
Number(function(){}) // NaN
Number({}) // NaN
Number('123abc') // NaN
Number(Math) // NaN
```

### String包覆器
`String()`如果傳入`eval()`函式的話，要特別注意。 `eval()`會把`String`物件視作是一個物件包含字串，而非一個字串。 所以要使用`valueOf()`或是`toString()`將`String`物件轉換成原始型別字串。

``` javascript
eval('5 + 5') // 10
eval(new String('6 + 6')) // 6 + 6
eval(new String('7 + 7').toString()) // 14
eval(new String('8 + 8').valueOf()) // 16
```

簡單一點，其實可以多加上原始型別字串，這樣也能達到一樣的效果。

``` javascript
eval(new String('8 + 8') + '8') // 24
```

## 結論
- 使用`new`運算子，在`typeof`的結果都會是`object`；反之，都會是自身的原始型別(`boolean`、`number`、`string`)。
- 使用`Boolean()`時，不要用`new`運算子。
- `Boolean()`和`!!`運算子功能相似，目的是要將物件轉換成布林值。
- `Number()`主要功能是將可轉換成數值的物件轉換成數值，否則回傳`NaN`物件。
- `Number()`可將`Date`物件轉換成時間戳記。
- `String()`在使用時，要注意在`eval()`中，需要做原始型別字串的轉換，否則會達不到預期的結果。