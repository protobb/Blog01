---
layout: post
title:  "Firebase 小小心得"
image: /public/res/firebase.png
---
![](/public/res/firebase.png)
Firebase 目前最新的版號是10.2.0  
稍微記錄一下使用心得

<!-- more -->

## 導入 ##
由於跟 Android Studio 深度整合，直接於工具列開始導入專案就好，幾乎不用做啥複雜的設定，他會全部裝好，還有簡單的 demo code

## Notification ##
官方已經明確表示要用FCM取代GCM，而且事實上FCM的程式碼也比GCM簡單很多，沒理由不換成FCM。

- 程式碼簡化：沒記錯的話只要實作兩個 class ,一個用來收 registration id，另一個用來顯示通知，就這樣。 
- FCM多了`topic` 群組的設計，如果有分群發信的需求，後端可以少一點工
- 官方提供的範例json長這樣

{%highlight json%}		
{ 
	"to":"topic/"
	"data":{DATA_OBJECT},
	"notification": {
		"title": "標題",
		"body": "主體"
	}
}
{%endhighlight%}

 但其實我實際使用起來，`Notification`這個區塊是不建議使用的，用了之後他似乎會觸發 firebase 自己實作的通知，所以你關不掉他XD 請養成習慣所有的東西都在`data`區塊傳送

## Crash Reporting ##
串 firebase 就這功能最讓我驚艷，因為你只要在 gradle 有 import ，那基本上它就是已經裝好了，完全不用寫一行 code ，超強XD  

後台會記錄基本的資料，像是手機型號、電信業者、當下記憶體之類的，如果有串 Analytics ，他會一併顯示當機前的幾個事件，方面你追蹤他來到這個畫面前的行為。  
如果需要手動回報你在 try-catch 區塊撈到的例外，可以這樣下:

{%highlight java%}
FirebaseCrash.report(new RuntimeException("noo~"));
{%endhighlight%}
那他在後台 Console 會被標記成不嚴重問題，相反的如果是造成閃退的就會被標記成嚴重。  

值得一提的是如果有執行程式碼模糊，它支援上傳 mapping 文件，幫你自動反模糊，方便！

## Analytics ##
Google 似乎也是希望 app 不要套 GA 了，改套這個，但我實際使用起來，覺得他的後台 console 很難用，相對於 GA 似乎精簡了很多東西。

而且最重要的是你沒辦法看單一 Event 所夾帶的資料，例如**加入購物車**這樣的事件，你可以看加了多少次，但你帶的**商品編號**卻看不到=.=，要看那些資訊似乎只能把它連動到[BigQuery](https://cloud.google.com/bigquery/)的服務了，這部份似乎要另外收費(?)  

相較之下我比較喜歡前公司 iOS 同事推薦的 [Mixpanel](https://mixpanel.com/) ，但唯一的問題是它要錢XD

## Test Lab ##
傳apk檔給他，他會幫你在雲端跑 testing 。  
就算沒有寫測試，它也可以幫你在畫面上找可以按的物件亂測一通，但我實測起來效果沒有很好，看來還是要乖乖寫測試才行啊XDD  

最後給你的測試報告包含所有噴過的log、螢幕錄影，還有trace過哪些畫面的樹狀圖跟每個畫面的截圖。


----------
以上，真的只是小小心得，畢竟真正精華的東西都還沒用過，像是Real-time Database，還有a/b testing之類的東西，等哪天有用過再來補充XD