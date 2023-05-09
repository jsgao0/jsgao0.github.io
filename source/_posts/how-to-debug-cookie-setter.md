---
title: 運用 `Object.defineProperty` 對 `document.cookie` 除錯
tags:
  - Cookies
  - JavaScript
  - debug
date: 2023-04-28 21:59:00
---

前陣子陸續發生兩次因為不合法的 cookie 值，在呼叫 API 時帶入到 HTTP request header 中，導致 API 在解析 cookie 值的時候發生錯誤，間接導致多數功能出現異常。
這篇文章會紀錄：
1. 如何利用 `Object.defineProperty` 和 `console.trace` 來攔截並查看是由誰來寫入 `document.cookie`
2. 延伸在 `Object.defineProperty` 的 `set` method 中，加入 cookie 值的 validation rule，避免非法的值被寫入其中，例如： `,` 或是 `"`。

## 查看 cookie 的值由哪一段 script 寫入

``` javascript
  function debugAccess(obj, prop) {
    var origValue = obj[prop];
    Object.defineProperty(obj, prop, {
      set: function(val) {
        /**
         * @note 這邊本來是用 `debugger` 來下中斷點，我改成用 `console.trace` 查看是由哪段 script 執行。
         */
        console.trace(val);
        return origValue = val;
      }
    });
  };
  debugAccess(document, 'cookie');
```

## 在攔截器中檢驗 cookie 的值

``` javascript
  function interceptCookie() {
    var origDescriptor = Object.getOwnPropertyDescriptor(Document.prototype, 'cookie');
    Object.defineProperty(document, 'cookie', {
      enumerable: true,
      configurable: true,
      get: function() {
        return origDescriptor.get.call(this);
      },
      set: function(val) {
        /**
           * @note cookie 的值會長這樣： `_the_cookie=the_value; domain=example.com`
           * 1. 開頭會是 `token=value` 這個結構
           * 2. 接著會用 `;` 或是 `; ` 接著其他選填的參數，詳情可參考 [MDN 說明](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie#write_a_new_cookie)
           * 
           * 所以在這邊會先取出第一個 `;` 的左邊區塊，將 `token=value` 開頭擷取出來；接著再取出 `=` 右邊的 `value`。
           */
        var cookieValue = val.split(';')[0].split('=')[1];

        /**
           * @note MDN 有寫道：
           * > The cookie value string can use encodeURIComponent() to ensure that the string does not contain any commas, semicolons, or whitespace (which are disallowed in cookie values).
           * 
           * 所以 `value` 若出現 `;`、`,`，或 ` `(空白) 的話，就不合法。因此在這邊用 Regex 檢查是否有包含這三個特殊字元，若有出現任一個的話則返回空值、不讓其寫入。
           * 值得一提的是， 邏輯上來說，這邊不需要再次檢驗 `;` 是否存在，因爲在解析 `cookieValue` 時，取出的值已經不可能包含 `;`了。
           */
        if (/[;,\s]+/.test(cookieValue)) {
          console.error('invalid cookie: ' + val);
        } else {
          origDescriptor.set.call(this, val);
        }
      }
    });
  }

  interceptCookie();
```

## 結論與心得
在現實世界中，任何事情都有可能發生。在做資料的序列化時，很容易會出現發送端 serialize 完的值送給接收端後，接收端做完 deserialize 後卻發現值有問題。所以在用一些比較特殊的 serializer 時，必須要看清楚序列化的文件才是。

## 參考資源
這裡使用 `Object.defineProperty` 作為攔截器的方法，是參考 [Breaking JavaScript execution when cookie is set](https://stackoverflow.com/questions/41245885/breaking-javascript-execution-when-cookie-is-set) 的[這個答案](https://stackoverflow.com/a/41247745)提供的範例和說明。