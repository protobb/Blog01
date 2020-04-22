---
layout: post
title: 對Android APP內的WebView進行偵錯
image: /public/res/2018-09-05_215920.jpg
description: 紀錄一下從怎麼從WebView內找出前端的問題並把它解決
---

![](/public/res/2018-09-05_215920.jpg)
早上收到PM訊息，說驗收時發現APP內的WebView沒有畫面   
我打開也確實發現是一片空白   

最麻煩的就是這種問題，完全沒有任何提示....

<!-- more -->

### 嘗試讀取錯誤訊息
隨便估狗了一下，發現原來可以直接用電腦版的Chrome去對手機APP內的Chrome偵錯，這應該就是我需要的東西了。

1. 配置程式
    ```java
    WebView.setWebContentsDebuggingEnabled(true);
    ```
    對你的Webview呼叫此方法，即可允許從電腦版去讀取手機的Chrome資訊了
2. 手機用USB接上電腦，然後確定USB偵錯啟用了 (身為一個Android Developer這些應該都設定好了)
3. 開啟你的Chrome，然後在網址列輸入 `chrome://inspect`  
    一切順利的話即會在畫面中顯示你的WebView的標題，點選**inspect**即可啟用開發者工具。

### 召喚前端工程師
進了開發者工具之後就是前端工程師的工作了。  
那如果你跟我一樣很不幸的也是那個前端工程師的話，你最後應該要可以從 console 跟一堆 js 檔裡面找出問題的癥結點。

我最後查到的原因是因為 WebView內的`localstorage`不能用，所以噴了NPE

### 正確解決問題
對WebView設定允許使用`localstorage`即可解決問題：
```java
WebView webView = (WebView) view.findViewById(R.id.webview);
webView.getSettings().setJavaScriptEnabled(true); //允許使用JS
webView.getSettings().setDomStorageEnabled(true); //允許使用localstorage
```
就這樣，客戶又可以繼續走驗收流程，我離結案以及尾款又更近一步了，真是可喜可賀。


## 參考資料
+ 遠程調試 WebView : [https://developers.google.com/web/tools/chrome-devtools/remote-debugging/webviews?hl=zh-tw](https://developers.google.com/web/tools/chrome-devtools/remote-debugging/webviews?hl=zh-tw)
+ Android webview & localStorage : [https://stackoverflow.com/questions/5899087/android-webview-localstorage](https://stackoverflow.com/questions/5899087/android-webview-localstorage)

<script type="application/ld+json">
{
    "@context": "http://schema.org",
    "@type": "HowTo",
    "name": "使用Google Chrome對Android APP內的WebView進行偵錯",
    "step": [
        {
            "@type": "HowToStep",
            "name": "配置程式碼",
            "text": "對你的Webview呼叫setWebContentsDebuggingEnabled()方法"
        },
        {
            "@type": "HowToStep",
            "name": "啟用USB偵錯",
            "text": "將手機連上電腦，並確定開發者模式內的USB偵錯已經啟用"
        },
        {
            "@type": "HowToStep",
            "name": "啟動Chrome",
            "text": "於Chrome的網址列輸入chrome://inspect，即可進入控制台"
        }
    ]
}
</script>