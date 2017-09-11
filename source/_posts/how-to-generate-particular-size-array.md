---
title: 如何產生特定長度的陣列
tags: 
    - JavaScript
    - ECMA262 6.0
    - Array
    - fill()
date: 2017-09-09 21:59:01
---

## 說明
在某些情況下，會需要初始化陣列，初始化包含兩個部分：指定長度、塞入初始值。常見的做法，不外乎就是用`for`或`while`來實作。
``` javascript
var i = 0, size = 4, arr1 = [], arr2 = [];
for(; i < size; i++) {
    arr1.push(i + 1);
}

i = 0;
while(i < size) {
    arr2.push(i + 1);
    i++;
}

console.log(arr1);
// [1, 2, 3, 4]

console.log(arr2);
// [1, 2, 3, 4]
```

## 使用方法
這裡要來分享怎麼用[Array.prototype.fill()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)簡短地初始化陣列。
``` javascript
var arr = new Array(4).fill(1).map(function(value, index) {
    return value + index;
});

console.log(arr)
// [1, 2, 3, 4]
```

### 問題
你可能會問：幹嘛要用fill來做這件事？ 直接在`new Array`後直接用`map()`塞值就好了啊！

### 回答
我的回答：因為用`new Array`建立的陣列，它並非我們預測的，會將每個位置都放入`undefined`，反倒是定義了陣列的長度而已。因此，它根本不會去迭代，想當然爾，就不會執行`map()`了。

### 驗證
``` javascript
var arr = new Array(4),
    filledArr = new Array(4).fill(1);

console.log(arr.hasOwnProperty(0))
// false

console.log(filledArr.hasOwnProperty(0))
// true
```

## 參考資料
[Array.prototype.fill() - MDN](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)
[Object.prototype.hasOwnProperty() - MDN](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)
[JavaScript “new Array(n)” and “Array.prototype.map” weirdness - StackOverflow](https://stackoverflow.com/questions/5501581/javascript-new-arrayn-and-array-prototype-map-weirdness)