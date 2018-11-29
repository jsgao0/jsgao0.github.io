---
title: 微信小程序與webview互動完成支付
tags: 
    - 微信
    - 小程序
    - web-view
    - 支付
    - WeChat
    - miniprogram
    - payment
date: 2018-11-29 21:26:00
---

## 緣由
敝公司在前陣子與大陸某公司合作，合作方式是讓該公司能在他們開發的微信小程序中，以 **web-view** 將敝公司的電商網站嵌入進去。 然而，小程序環境裡面，只能夠使用小程序支付的 API 。 因為這個原因，才想到了這樣子的支付方式。

## 流程簡述
I. 建立一個小程序支付頁面，頁面名稱為 `payment-page` 。

II. 在 **webview頁面** 裡面，引入微信的 [JSSDK](https://res.wx.qq.com/open/js/jweixin-1.3.2.js) 後，使用下面語法將[文件](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_7&index=5)提到的一些參數帶入小程序頁面連結，並且跳轉到小程序支付頁面(`payment-page`)。


``` javascript 
    wx.miniProgram.navigateTo({
        url: '/payment-page?paySign=xxx&timeStamp=yyy&nonceStr=111&package=222&signType=MD5'
    });
```

III. 在小程序支付頁面(`payment-page`)的 `onLoad(Object query)` 事件函式裡面，從回呼的參數 `query` 取出 `paySign` 、 `timeStamp` 、 `nonceStr` 、 `package` 、 `signType` 。

IV. 同樣在小程序支付頁面(`payment-page`)的 `onLoad` 函式內，使用前一步驟取出的五個參數帶入 `wx.requestPayment` 作為呼叫的參數，同時加上 `success` 、 `fail` 、 `complete` 等三個 `callback` 函式，作為支付回呼的處理之用。


``` javascript
    wx.requestPayment({
        'paySign': '12345678dlksfjldfj',
        'timeStamp': 'xxx',
        'nonceStr': 'yyy',
        'package': 'nonceStr',
        'signType': 'MD5',
        'success': function(res){},
        'fail': function(res){},
        'complete': function(res){}
    });
```

## 坑
1. `postMessage` 只能觀賞用
一開始打算直接透過 `postMessage` 將我們自己後端回傳的資料發給小程序，但是，事情不是憨人所想的這麼簡單。
<blockquote>
  网页向小程序 postMessage 时，会在特定时机（小程序后退、组件销毁、分享）触发并收到消息。e.detail = { data }
  <footer style="text-align: right;">https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html?search-key=postmessage</footer>
</blockquote>
嗯... 觀賞用。

2. `paySign` 是第二次簽名的產物
傻傻看到官方文件最上面的表，標題清楚寫著**小程序调起支付数据签名字段列表**。 一般人絕對會認為直接帶這幾個參數到 `wx.requestPayment` 就好，顆顆... 我錯了，要把文件看完才對。

<blockquote>
  <p>举例如下:</p>
  <p style="padding:10px; overflow:auto;">paySign=MD5(appId=wxd678efh567hg6787&nonceStr=5K8264ILTKCH16CQ2502SI8ZNMTM67VS&package=prepay_id=wx2017033010242291fcfe0db70013231072&signType=MD5&timeStamp=1490840662&key=qazwsxedcrfvtgbyhnujmikolp111111)=22D9B4E54AB1950F51E0649E8810ACD6</p>
  <p>详细签名算法请参考“[签名算法](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=4_3)”说明</p>
  <footer style="text-align: right;">https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_7&index=5</footer>
</blockquote>

## 小結
文件真的要看清楚，看完還必須親身去採坑，這樣才能夠親身體驗灰頭土臉的快感。 後面取得請求支付的參數那段就先不提，這邊只提網頁和小程序互動完成支付的部分。