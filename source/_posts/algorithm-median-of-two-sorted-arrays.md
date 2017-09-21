---
title: LeetCode - Median of Two Sorted Arrays
tags: 
    - Algorithm
    - LeetCode
    - Hard
    - JavaScript
date: 2017-09-21 22:14:45
---

## 題目說明
簡單來講，就是要取出兩個已排序過的陣列中，所有數字的中位數。 且，整體時間複雜度應該為`O(log(m+n))`。
例如：
``` javascript
var arr1 = [1, 2],
    arr2 = [3, 4];
    median(arr1, arr2); // 2.5
```

## 思考
說到中位數，第一個想到的是總數為奇數和偶數的計算方式並不同。
例如：
``` javascript
var arr1 = [4, 8],
    arr2 = [2, 5, 9];
    median(arr1, arr2); // 5，兩個陣列加起來共有五個數字，中位數會在第三個位置。
```

``` javascript
var arr1 = [5, 10, 25],
    arr2 = [2];
    median(arr1, arr2); // 7.5，兩個陣列加起來共有四個數字，中位數會是中間兩個數字的平均值，(5 + 10) / 2 = 7.5。
```

### 問題來了，我們要怎麼取得中位數？
中位數還有一個特點，它只需要用**索引**就能夠找到。 換句話說，我們可以先找出兩個陣列總數的中位數**索引**，接著，再去針對奇偶數做不同的計算。 一開始我本來打算用`concat()`將兩個陣列合併起來，直接取中位數。 但想想，儘管合併，還需要再做一次排序，這樣反而花更多時間。 然而，我想到另一個方法，用慢慢逼近中位數位置的方式進行。

### 找出索引值
在逼近中位數之前，我們一樣要先知道中位數的索引值。 假定總數為奇數的話，給定中位數索引值為(n + m + 1) / 2；若為偶數的話，給定中位數索引值為(n + m) / 2 + 1。 前者應該很好理解，後者則是為了要計算中間兩數平均而加一。
舉個例子：
``` javascript
var arr1 = [4, 8],
    arr2 = [2, 5, 9];
    medianIndex(arr1, arr2); // 3 = (2 + 3 + 1) / 2 
```

``` javascript
var arr1 = [5, 10, 25],
    arr2 = [2, 5, 17];
    medianIndex(arr1, arr2); // 4 = (3 + 3) / 2 + 1
```
### 慢慢逼近中位數
既然我們知道中位數的索引值，我們可以慢慢地從兩個陣列尾端慢慢將當前最大的數字抽出，直到總數跟其索引值相同。
舉個例子：
``` javascript
var arr1 = [4, 8],
    arr2 = [2, 5, 9],
    medianIndex = 3;
    pop(arr1, arr2); // total = 4, arr1 = [4, 8], arr2 = [2, 5]
    pop(arr1, arr2); // total = 3, arr1 = [4], arr2 = [2, 5]
```

``` javascript
var arr1 = [5, 10, 25],
    arr2 = [2, 5, 17],
    medianIndex = 4;
    pop(arr1, arr2); // total = 5, arr1 = [5, 10], arr2 = [2, 5, 17]
    pop(arr1, arr2); // total = 4, arr1 = [5, 10], arr2 = [2, 5]
```

### 取出或計算中位數
你現在應該有發現到，總數若為奇數的話，把兩個陣列中最大的數字取出來即為中位數；總數若為偶數的話，則多取第二大的數字，把它跟最大的數字取平均值，即為中位數。
``` javascript
var arr1 = [4, 8],
    arr2 = [2, 5, 9],
    medianIndex = 3;
    pop(arr1, arr2); // total = 4, arr1 = [4, 8], arr2 = [2, 5]
    pop(arr1, arr2); // total = 3, arr1 = [4], arr2 = [2, 5]
    maxNumber(arr1, arr2); // 5
```

``` javascript
var arr1 = [5, 10, 25],
    arr2 = [2, 5, 17],
    medianIndex = 4;
    pop(arr1, arr2); // total = 5, arr1 = [5, 10], arr2 = [2, 5, 17]
    pop(arr1, arr2); // total = 4, arr1 = [5, 10], arr2 = [2, 5]
    maxNumbersAvg(arr1, arr2); // 7.5 = (5 + 10) / 2
```

## 程式碼
``` javascript
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number}
 */
var findMedianSortedArrays = function(nums1, nums2) {
    var len1 = nums1.length,
        len2 = nums2.length,
        total = len1 + len2,
        totalIsEven = (total % 2 === 0),
        medianIndex = totalIsEven ? (total / 2 + 1) : Math.ceil(total / 2), // 找出索引值
        median = 0;
  
    // 慢慢逼近中位數
    while(total > medianIndex) {
        if(nums1.length > 0) {
            if(nums2.length > 0) {
                if(nums1[nums1.length - 1] > nums2[nums2.length - 1]) {
                    nums1.pop();
                } else {
                    nums2.pop();
                }
            } else {
                nums1.pop();
            }
        } else {
            nums2.pop();
        }
        total--;
    }
    
    // 取出兩個陣列中的最大值
    if((nums1[nums1.length - 1] || Number.MIN_VALUE) > (nums2[nums2.length - 1] || Number.MIN_VALUE)) {
        median = nums1.pop();
    } else {
        median = nums2.pop();
    }
  
    // 如果是偶數，多取出兩個陣列中第二大的值
    if(totalIsEven) {
        if((nums1[nums1.length - 1] || Number.MIN_VALUE) > (nums2[nums2.length - 1] || Number.MIN_VALUE)) {
            median += nums1.pop();
        } else {
            median += nums2.pop();
        }
        median = median / 2;
    }
    
    return median;
    
};
```

### 偷懶的小區塊
你應該有發現到`nums1[nums1.length - 1] || Number.MIN_VALUE`這個東西怪怪的，幹嘛這樣做？ 這只是要避免陣列在逼近結束或取出最大值之後，變成空陣列而取出`undefined`，造成計算結果為`NaN`的偷懶做法而已ＸＤ。

## 結論
我根本來亂的... 照常理，我的算法時間複雜度應該是`O(n)`。 感覺可以再加入二分法再做一些優化，有時間再來好好改善一下。