---
title: 如何動態加入包含onload回呼函式的script元素
tags: 
    - JavaScript
    - ECMAScript
    - script element
    - onload
    - frameset
    - frame
date: 2017-09-11 22:33:15
---

## 說明
在下面的`html`範例中，包含了兩個`frame`，分別為**headerFrame**和**contentFrame**。 然而，我想要在這份`html`管理**contentFrame**的一些*js*的全域變數。 可是，我如果放在這份`html`文件的`head`裡面的話，`frame`裡面完全抓不到這些全域變數。 除此之外，如果我要在這些檔案被載入的時候，執行一些自訂的函式又要怎麼做？

``` html
<!DOCTYPE>
<html>
    <head>
       ...
    </head>
    <body>
        <frameset>
            <frame name="headerFrame"></frame>
            <frame name="contentFrame"></frame>
        </frameset>
    </body>
</html>
```

## 作法
一開始，我很單純地以為用`document.createElement('script')`就好，沒想到卻沒這麼簡單。 我一共遇到了3個問題：
* 怎麼樣取得**contentFrame**裡頭的`document`?
* 怎樣把建立好了`script`加到該`document`裡面？
* 如何綁定`onload`的回呼函式，且支援IE7？

### Step 1
要怎麼取得`frame`裡面的`document`物件？ 先在`window`物件的`frames`屬性把**contentFrame**取出來，再將這個`frame`的`document`取出來。
``` javascript
var frameDoc = window.frames['contentFrame'].document;
```

### Step 2
接著，怎麼樣把`script`加到`document`? 先用`document.createElement()`創建一個新的`script`元素，並設定這個元素的`src`和`type`屬性。 然後用`frameDoc`物件(**contentFrame**裡面的`document`)的`head`屬性提供的`appendChild()`方法將元素加到`head`裡面去。
``` javascript
var frameDoc = window.frames['contentFrame'].document,
    newScript = document.createElement('script');
newScript.src = '/js/boswer.min.js';
newScript.type = 'text/javascript';
frameDoc.head.appendChild(newScript);
```

這邊有另外一個小問題，`document.head`這個屬性在[**IE 9**](http://caniuse.com/#feat=documenthead)以後才開始支援。 所以要做個簡單的填充工具，利用`document.getElementsByTagName()`把`head`加到`document`中。
``` javascript
var frameDoc = window.frames['contentFrame'].document,
    newScript = document.createElement('script');
newScript.src = '/js/boswer.min.js';
newScript.type = 'text/javascript';
if(!frameDoc.head) frameDoc.head = frameDoc.getElementsByTagName('head')[0];
frameDoc.head.appendChild(newScript);
```

### Step 3
一般來講，`onload`會是`script`元素的一個屬性，只需把`function`指派給這個物件即可。
``` javascript
var frameDoc = window.frames['contentFrame'].document,
    newScript = document.createElement('script');
newScript.src = '/js/boswer.min.js';
newScript.type = 'text/javascript';
newScript.onload = _onload;
if(!frameDoc.head) frameDoc.head = frameDoc.getElementsByTagName('head')[0];
frameDoc.head.appendChild(newScript);

function _onload() {
    // script被載入之後才會執行的程式碼
}
```

問題又來了，**IE 9**之後才開始支援[**DOMContentLoaded**](http://caniuse.com/#search=onload)。 因此，可以採用`script.onreadystatechange()`作為舊版本**IE**的替代方案。
``` javascript
var frameDoc = window.frames['contentFrame'].document,
    newScript = document.createElement('script');
newScript.src = '/js/boswer.min.js';
newScript.type = 'text/javascript';
newScript.onload = _onload;
newScript.onreadystatechange = function () {
    if (this.readyState == 'complete') _onload();
}
if(!frameDoc.head) frameDoc.head = frameDoc.getElementsByTagName('head')[0];
frameDoc.head.appendChild(newScript);

function _onload() {
    // script被載入之後才會執行的程式碼
}
```

## 參考資料
[document.head - caniuse.com](http://caniuse.com/#feat=documenthead)
[onload - caniuse.com](http://caniuse.com/#search=onload)
[Object.onload in Internet Explorer 6, 7 and 8 - StackOverflow](https://stackoverflow.com/questions/6806584/object-onload-in-internet-explorer-6-7-and-8)