---
layout: post
title: 從無到有把公司網站架起來
description: 紀錄一下從架主機開始到網站上線到底做了哪些事情
image: /public/res/cyberspace-2784907_640.jpg
---
![](/public/res/cyberspace-2784907_640.jpg)

前前後後耗了三個禮拜終於趕在公司發表活動前把官網架起來了

做為公司僅存的網頁工程師，除了遇到問題沒人可以救我之外，東西架好了也沒人交接呢。所以我決定簡單記錄一下這陣子到底做過哪些事情，以免以後又要浪費時間重新估狗一次

<!-- more -->

# 0. 簡介
公司是微軟愛用者，所以用了整套的Windows Solution全餐……
+ Windows Server 2016
+ IIS 7
+ MS SQL
+ .NET Framework

本篇不會談到SQL Server的部分，那是我唯一有同事幫忙處理的東西，在此默默感謝同時身兼PM跟DBA的妹子同事，雖然她應該看不到這篇文章。

# 1. 硬體
主要是要記得開防火牆跟設定Router映射，就不浪費篇幅討論怎麼安裝Windows Server跟IIS了，~~其實主機從前公司幹來的拿到的時候已經裝好了~~

## 防火牆
http 走 80 port，而 https 走 443，這兩個port要開，我想會來看這篇的人這個應該是基本常識。
## Router映射
映射指的是當你打外部IP跟port號的時候，由Router幫你轉到內部IP的某個port，例如

    XXX.XXX.XXX.XXX:80  -> 192.168.0.YY:80
    XXX.XXX.XXX.XXX:443 -> 192.168.0.YY:443

這個部分要在你的路由器上面設定。  
所以，除了你的公司要用固定IP之外，你拿來架站的伺服器內部也是要配靜態IP，不要傻呼呼的用DHCP自動配發，不然他哪天就消失在網路上了。

# 2. 程式
## 開發
身為一個網頁工程師，把 UI Designer 設計的畫面變成一個.NET專案應該是沒啥好紀錄的。總之就是先在外面用熟悉的前端開發工具(VSCode?)，把html css js檔寫完，然後通通扔到專案裡面就好了呢。
## 部署
有 Gitlab CI，上版輕輕鬆鬆毫無煩惱
> 前情提要：[C#.NET專案使用Gitlab CI部署至IIS](/2018/04/14/csharp-gitlab-ci.html)

# 3. 連上網路
## 域名
公司域名託管在[Hinet](https://domain.hinet.net/)上，反正就登進去把域名指向公司固定IP就好。

## SSL憑證
現在的網站要是沒有走HTTPS，不僅會被Chrome標上大大的**不安全**、搜尋引擎會調降你的排名之外，最重要的是公司的工程師會被認為**技術力很廢很沒用**，這個我不能忍，所以就自己默默把憑證搞好了。
![](/public/res/JqouljA.jpg)
> 網站不安全我可以忍，但質疑我就不行

### 申請憑證
此部分使用簡單易用的[SSL For Free](https://www.sslforfree.com/)服務，幫你搞定Let's Encrypt的大小事情。

1. 於網站上輸入公司域名(多個域名用空格隔開)
2. 選擇手動認證，但在那之前需要先處理`.well-known\acme-challenge`問題:
    + windows無法使用`.`字頭的資料夾，此部分可使用IIS內的**虛擬目錄**的功能，將其指向伺服器上的其他目錄，同時還可以迴避因為CI重新建置導致資料夾被刪除的問題。
    ![](/public/res/asofijfoij.jpg)
    + IIS還須設定`MIME/Type`將`.`對應到`text/plain`才能正確使用。
3. 將網站上要你下載的檔案們放到`.well-known\acme-challenge`底下並且確定可以被公開讀取，他會透過讀取此檔案來確認網站是你架的
    > HINT:注意實體資料夾的權限問題
4. 以上皆沒出錯的話，網站會告訴你成功驗證，並且會要你下載憑證，就乖乖把憑證下載下來吧，裡面應該包含三個檔案:
    + `private.key`
    + `ca_bundle.crt`
    + `certificate.crt`
5. 於同一個畫面輸入email，他會在憑證快要過期的時候通知你，十分貼心

到此憑證申請完成，但是還不能用...  
由於我們親愛的IIS只吃附檔名為`.pfx`的憑證，所以還需要轉換一下。

### 憑證轉檔

1. 準備工具 [OpenSSL](http://slproweb.com/products/Win32OpenSSL.html)
2. 使用以下指令轉換
```
openssl pkcs12 -export -out YOUR\FOLDER\certificate.pfx -inkey YOUR\FOLDER\private.key -in YOUR\FOLDER\certificate.crt -certfile YOUR\FOLDER\ca_bundle.crt
```
3. 產出過程會要你輸入密碼，照做就對了

### 憑證安裝
1. 於IIS最高層的那個設定畫面的`Server Certificate`設定把剛剛生成的`.pfx`檔餵進去
2. 回到站點層的設定，將https跟443埠號binding起來，並在那個設定畫面選擇你剛剛匯入的憑證。
![](/public/res/iis_443_binding.jpg)

至此大致完成，但要注意一個地雷:  
站點層的設定有一個是`SSL Setting`，那個其實根本次設定完全無關，請記得維持`Ignore`的設定，不然瀏覽網頁時，使用者會被要求提供憑證

### 強制導向
辛辛苦苦設定好安全連線，但使用者還是可能會傻呼呼的使用HTTP通道連線，然後再跑來問你為啥估狗說你家的網站不安全？於是我們需要強制轉址。

此功能仰賴IIS的Rewrite功能完成實作，步驟如下；
1. 安裝 Rewrite
    > 下載網站：[URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
2. 在全站的`Web.config`中，找到`<system.webServer>`區塊
3. 區塊內新增以下程式碼:
{%highlight xml%}
<rewrite>
  <rules>
    <rule name="HTTP/S to HTTPS Redirect" enabled="true" stopProcessing="true">
    <match url="(.*)" />
    <conditions logicalGrouping="MatchAny">
      <add input="{SERVER_PORT_SECURE}" pattern="^0$" />
    </conditions>
    <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" redirectType="Permanent" />
    </rule>
  </rules>
</rewrite>
{%endhighlight%}

以上，就會自動生效了。
> 以上轉址的部分參考自:[此網站](https://blog.miniasp.com/post/2014/06/04/Redirect-to-HTTPS-from-HTTP-using-IIS-URL-Rewrite.aspx)

# 4. 上線之後
如果是客戶的專案，網站上線之後任務就結束了，但現在是自家公司的東西，所以還有一些事情可以做得更好：
## 成效追蹤
一切仰賴估狗大神的[Google Analytics](https://www.google.com/analytics/)，網站寫得夠清楚了，進去一步一步照著把程式碼貼一貼，就初步搞定了。
> 或許也可以裝個Facebook Pixel

## 搜尋引擎優化(SEO) 
此部分主要是去[Search Console](https://www.google.com/webmasters/tools/home?hl=zh-TW)註冊你們家的網站，用來確認估狗的搜尋引擎可以正確爬你們家的網站
## 網站性能優化
又是估狗的工具: [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)  
來這邊跑一遍，他會給你很多優化建議，例如快取啦，靜態檔案壓縮之類的，他甚至會幫你壓好所有優化過的檔案讓你下載，值得一試的網站

**靜態檔案快取**  
如果你的檔案有放在同一個資料夾，可以在IIS左側樹狀欄位選到該資料夾之後從右側選擇`Http Response Header`設定，進去後右上角有個`Set Common Headers`可以直接指定Cache過期時間

**壓縮**  
iis可以安裝`dynamic compression`組件幫你完成這個任務
## 分享偵錯
社群界的霸主提供的工具:[分享偵錯工具](https://developers.facebook.com/tools/debug/sharing/)  
該工具用來預覽你的網站貼到FB之後縮圖會長什麼樣子，由於這套規範(Open Graph)就是由FB主導定義的，所以用FB的工具來測試自然就是最準的啦。


# 5. 後記
其實有很大一個重點沒有提到，就是為啥架個靜態網站會需要有人幫我弄SQL Server……   
真正的原因是公司網站同時也提供API供合作廠商介接，但總覺得這部分再寫下去會寫不完，所以就整個跳過好了，以後用空再另外些一篇文章來聊這塊。

當一個從頭架到尾的真．全端工程師真的很幹，但是其實搞定之後的成就感也是挺不錯的，看來離可以自己接案又更進一步了呢 (ﾟ∀。)