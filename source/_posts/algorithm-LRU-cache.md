---
title: LeetCode - LRU Cache
tags: 
    - Algorithm
    - LeetCode
    - Hard
    - JavaScript
date: 2017-10-04 17:57:00
---

## 題目說明
簡單的說，就是建立一個快取的資料結構，它能夠在 `get(key)` 和 `put(key, value)` 的時候，調整快取的內容。 在調用 `get(key)` 函式的時候，找到對應的值，回傳該值；若在快取中找不到值，回傳 `-1` 。 另一方面，在調用 `put(key, value)` 的時候，會將其值加到快取中，若已經存在 `key` 對應的值，將其值更新；若在快取中找不到值且快取已滿，將最久沒使用到的值從快取中剔除，並將加入新的值到快取。 嘗試將 `get(key)` 和 `put(key)` 以時間複雜度為 `O(1)` 的方式實作。
例如：
``` javascript
var lru = new LRUCache(3); // 最多存放3的值
lru.put(4, 40); // 快取: 4(40)
lru.put(1, 10); // 快取: 4(40), 1(10)
lru.get(3);     // 快取: 4(40), 1(10) ; 回傳 -1
lru.get(1);     // 快取: 4(40), 1(10) ; 回傳 -1
lru.get(2);     // 快取: 4(40), 1(10) ; 回傳 -1
lru.get(4);     // 快取: 1(10), 4(40) ; 回傳 40
lru.put(3, 30); // 快取: 1(10), 4(40), 3(30)
lru.put(2, 20); // 快取: 4(40), 3(30), 2(20)
```

## 思考
乍看之下，用一個 `Array` 搭配 `Array.prototype.find()` 應該就可以解決。 但 `Array.prototype.find()` 的時間複雜度是 `O(n)` ，並不是我要的 `O(1)` 。 於是乎，去 `LeetCode` 的討論區爬到了一個解法，用 [`HashTable + DoubleLinkedList`](https://discuss.leetcode.com/topic/6613/java-hashtable-double-linked-list-with-a-touch-of-pseudo-nodes/23) 實作。

### 雜湊表 - HashTable
雜湊表是一個映射表，可以透過唯一的鍵值(key)，查詢表中所對應的資料(value)。 然而，因為是一個鍵值對應的關係，在查找的時候，時間複雜度只需要
 `O(1)`。 因此，採用雜湊表的目的，是要來管理目前快取裡面存在的值，無論新增、修改、刪除都能夠以 `O(1)` 的時間複雜度完成。 


### 雙向連結串列 - Double Linked List
而雙向連結串列的用途，是作為時序(順序)管理，利用雜湊表快速找到對應的節點，直接以該節點的前、後作為插入或刪除的依據。

### put(key, value) 函式
在把節點加入到快取的時候，有可能該節點所包含的鍵值已經存在於雜湊表中，所以也有兩種情況要考慮：

1. 找到對應的值 -> 更新節點的值
2. 找不到對應的值 -> 將節點加入到快取中

然而，因為快取有容量的限制，我們在容量已滿的情況下，應該把快取中最久沒被存取的節點刪除，再將新的節點加至快取中，並且視為是最新的節點。

下面的例子中，在執行 `lru.put(3, 30);` 後，快取的容量已經滿了。 所以在接下來的 `lru.put(6, 60);` 執行之後， `4(40)` 會被擠出快取，即從快取中被刪除。 

另一方面，如果是更新原有的節點，該節點會被挪至最新的位置。 `lru.put(1, 20);` 執行後， `1(20)` 會取代掉 `1(10)` ，並且被挪至 `6(60)` 的後面，即時序中最新的位置。
``` javascript
var lru = new LRUCache(3); // 最多存放3的值
lru.put(4, 40); // 快取: 4(40)
lru.put(1, 10); // 快取: 4(40), 1(10)
lru.put(3, 30); // 快取: 4(40), 1(10), 3(30)
lru.put(6, 60); // 快取: 1(10), 3(30), 6(60)
lru.put(1, 20); // 快取: 3(30), 6(60), 1(20)
```

### get(key) 函式
從快取中搜尋並取得對應的值，可能會有兩種情況：

1. 找到對應的值 -> 回傳該值
2. 找不到對應的值 -> 回傳-1

此外，還需要對快取的內容做時序上的調整，將當次的值視為是最新存取的。 例如下面的範例，在第二次執行 `lru.put(1, 10);` 的時候，快取的時序為 `4(40), 1(10)` ，意即 `4(40)` 較為 `1(10)` 來得舊。
``` javascript
var lru = new LRUCache(3); // 最多存放3的值
lru.put(4, 40); // 快取: 4(40)
lru.put(1, 10); // 快取: 4(40), 1(10)
lru.get(4);     // 快取: 1(10), 4(40) ; 回傳 -1
```

### 實作1 - LRUCache 建構式
因為 *JavaScript* 的物件本身就有 *(key, value)* 的特性，所以雜湊表的部分我直接以一個 `Object` 取代，然後再自己建立雙向連結的節點類別。
``` javascript
var hashTable = {}; // 雜湊表

function DoubleLinkedNode(key, value) { // 雙向連結的節點類別
    this.key = key;
    this.value = value;
    this.prevNode;
    this.nextNode;
}
```

再來， `LRUCache` 建構式需要能傳入快取的容量，以便在建立快取實體的時候，限制快取的大小。 
``` javascript
var LRUCache = function(capacity) {
    this.capacity = capacity; // 快取容量
}
```

裡面還需要一個計數器，去紀錄當前快取的大小。
``` javascript
var LRUCache = function(capacity) {
    this.capacity = capacity;
    this.size = 0; // 當前容量的計數器
}
```

雜湊表也要記得丟進去！
``` javascript
var LRUCache = function(capacity) {
    this.capacity = capacity;
    this.size = 0;

    this.hashMap = {}; // 雜湊表
}
```

目前為止，容量指標、雜湊表都有了，接著是雙向連結的串列。 在雙向連結串列中，我們需要它的頭跟尾，分別作為時序中 **最久的節點** 和 **最新的節點** 。 
``` javascript
var LRUCache = function(capacity) {
    this.capacity = capacity;
    this.size = 0;

    this.hashMap = {}; // 雜湊表

    this.head = new DoubleLinkedNode();
    this.head.prevNode = null;
    this.tail = new DoubleLinkedNode();
    this.tail.nextNode = null;
    this.head.nextNode = this.tail;
    this.tail.prevNode = this.head;
}
```

除此之外，還需要一些雙向連結串列的操作式：

1. appendNode - 加入最新的節點
2. removeNode - 刪除節點
3. moveToTail - 將現有節點移到最新的位置(尾巴)
4. peekHead   - 移除最舊的節點(頭)

**注意：頭跟尾的處理，並非真的刪除頭、新增節點到尾巴後面，而是處理最靠近頭跟尾的節點。**

``` javascript
var LRUCache = function(capacity) {
    this.capacity = capacity;
    this.size = 0;

    this.hashMap = {}; // 雜湊表

    this.head = new DoubleLinkedNode();
    this.head.prevNode = null;
    this.tail = new DoubleLinkedNode();
    this.tail.nextNode = null;
    this.head.nextNode = this.tail;
    this.tail.prevNode = this.head;

    this.appendNode = function(node) {
        node.nextNode = this.tail;
        node.prevNode = this.tail.prevNode;

        this.tail.prevNode.nextNode = node;
        this.tail.prevNode = node;
    };

    this.removeNode = function(node) {
        var prevNode = node.prevNode,
            nextNode = node.nextNode;
        prevNode.nextNode = nextNode;
        nextNode.prevNode = prevNode;
    };

    this.moveToTail = function(node) {
        this.removeNode(node);
        this.appendNode(node);
    };

    this.peekHead = function() {
        var retNode = this.head.nextNode;
        this.removeNode(retNode);
        return retNode;
    }
}
```

### 實作2 - put函式
實作這個函式需要注意的地方，是

``` javascript
LRUCache.prototype.put = function(key, value) {
    var node = this.hashMap[key],
        newNode, headNode;

    if(node) { // 找到對應的值
        node.value = value; // 更新節點的值
        this.moveToTail(node); // 將節點移到最新的位置
    } else { // 找不到對應的值 
        newNode = new DoubleLinkedNode(key, value); // 建立新的節點
        this.hashMap[key] = newNode; // 把新的節點加到雜湊表
        this.appendNode(newNode); // 把新的節點加到雙向連結串列之中最新的位置(尾巴)
        this.size++; // 當前容量加1
        if(this.size > this.maxSize) { // 若當前容量超過快取最大容量
            headNode = this.peekHead(); // 把串列中最舊的節點(串列的頭)刪除
            delete this.hashMap[headNode.key]; // 把最舊的節點也從雜湊表刪除
            this.size--; // 當前容量減1
        }
    }
};
```

### 實作3 - get函式
``` javascript
LRUCache.prototype.get = function(key) {
    var node = this.hashMap[key]; // 用key查找雜湊表中對應的節點
    if(!node) { // 若找不到節點
        return -1; // 回傳-1
    }
    this.moveToTail(node); // 若有找到節點，把現有的節點移到最新的位置(串列的尾巴)
    return node.value; // 若有找到節點，回傳節點的值
};
```

## 結論
儘管這個方法是從討論區找到的，還是花了好一段時間理解這段程式。 在原始的範例中，是以串列尾巴作為最舊的節點、串列的頭作為最新的節點。 而我為了練習，把它反轉過來，把頭當最舊、尾當最新。 
除此之外，在串列的操作上，尤其 `appendNode` 和 `removeNode` 花了較多時間在理解做法。 主要是因為我的資料結構都忘光光的關係吧！ＸＤ
至於雜湊表，這邊只是拿來作為簡單的字典使用，不用真的算出雜湊值...

## 參考資料
- [LeetCode - 146. LRU Cache](https://leetcode.com/problems/lru-cache/description/)
- [雜湊表](https://en.wikipedia.org/wiki/Hash_table)
- [雙向連結串列](https://en.wikipedia.org/wiki/Doubly_linked_list)