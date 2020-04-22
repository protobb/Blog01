---
layout: post
title:  "來寫個bookmarklet吧"
description: 簡單介紹一下Bookmarklets的寫法與使用限制
---

前幾年某天深夜想對某個網站做一些操作，卻發現該站沒有使用jquery，那時候就在想有沒有一個可以快速注入jquery到網站內的方法，後來就找到這個東西- bookmarklet 書籤列小工具。

<!-- more -->

## 原理
他其實就是把javascript的程式碼存在書籤的網址列裡面，當你點擊書籤的時候就會執行網址列內的程式碼片段，藉此完成你預計完成的任務。

## 格式
實際在撰寫的時候，外層要額外包裹起來才能存成書籤執行，所以你的程式碼最後會長這樣：

```javascript
javascript: (function() {
    //your code
})()
```
我有在github上開一個repo放我寫過的小工具，可以參考看看  
[janelin612/bookmarklets](https://github.com/janelin612/bookmarklets){: target="_blank"}

## 限制
這東西肯定不是萬能的，需要注意以下幾點 
+ 長度：他被視為一段網址，長度受到瀏覽器的網址長度限制，因此在開發上就是要努力精簡程式碼啦。
+ [CORS](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS){: target="_blank"}：別忘了他還是在前端執行的，所以它一樣要遵守跨網域存取的規範，這其實大大限制了小工具能做的事情，但沒辦法嘛，安全性問題。

## 後記
+ 如果你是Firefox的使用者，可以使用[程式碼速記本](https://developer.mozilla.org/zh-TW/docs/Tools/Scratchpad){: target="_blank"}協助開發，比F12的主控台好寫(但你還是得開F12看console就是了)
+ 這玩意如果是惡意的用途，就算是一種[Self-XSS](https://en.wikipedia.org/wiki/Self-XSS){: target="_blank"}，所以拿到別人分享的小工具還是要先看一下程式碼有沒有問題不要傻傻地就執行下去了