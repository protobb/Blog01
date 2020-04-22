---
layout: post
title: 使用Windows IIS 架設 Wordpress
image: /public/res/wordpress-265132_1280.jpg
---

![](/public/res/wordpress-265132_1280.jpg)

公司需要一個CMS平台，主管叫我評估一下要自己寫還是用現成的工具  
那我當然是用現成的囉(攤手)

<!-- more -->

# 事前評估
主要考量的點還是公司沒幾個工程師(好啦其實就只有我一個)，不太可能自己開發，所以決定用Wordpress頂著用，搞不好一個不小心還可以把購物系統一起架在上面

我最初的規劃其實想要用docker架站  
但礙於公司遲遲不幫我生出一台Linux Server，最後只好委屈點，跟公司其他服務共用一台主機了。

# 安裝
## 1. 準備PHP環境
> 下載頁:[PHP](http://php.net/downloads.php){: target="_blank"}  

### 下載&安裝
選Windows版，然後就選最新穩定版就對了  
下載完成後解壓縮至自已喜歡的地方，我放`C:\php`  
放好之後別忘了:
1. 將該目錄加到環境變數內
2. 開放IIS對該資料夾的讀寫權限

### 設定
接著開始設定，資料夾根目錄內有兩個檔案  
分別是`php.ini-development`&`php.ini-production`，挑一個複製出來，重新命名成`php.ini`，他會成為你正式的設定檔。  
要異動的設定有下面幾個:
+ 732行 `extension_dir` 指定成你目錄內的`ext`資料夾，例如`C:\php\ext`
+ 749行 `cgi.force_redirect = 0`
+ 769行 `cgi.fix_pathinfo=1`
+ 782行 `fastcgi.impersonate = 1`
+ 886行開始有一系列的extension設定，我開了這幾個:
    + curl
    + mysqli
    + openssl
    + pdo_mysql

> 網路上很多教學都寫說設定檔要寫成`extension=php_openssl.dll`但新版的PHP不用這樣寫了，直接取消註解就好。

以上大概就完成PHP的設定了 :D

## 2. 安裝MySQL
> 下載頁:[Download MySQL Installer](https://dev.mysql.com/downloads/installer/){: target="_blank"}

mysql我不太熟悉，所以選擇直接用官方提供的wizard安裝  
解壓縮後就照著安裝步驟走，除了以下事情要注意之外都用預設值就好，這樣就可以順利完成安裝了
+ Authentication Type : 預設選項是`caching_sha2_password`，但我發現裝好之後會導致Wordpress無法順利連接mysql，所以又重新回來這邊設定，改成`Standard`

### 設定
如果上面都照我說的走，那你應該有MySQL Workbench這套GUI工具可以使用，就用它來完成後續的設定吧
1. 開一顆DB : 上方工具列選`Create a new schema`，然後命名為`wordpress`，Charset設成`utf8`後存檔
2. 建立使用者 : 左方工具選`Users and Privileges`,下方`Add Account`選下去，然後把帳號密碼設一設
> Authentication Type記得要選`Standard`
3. 給予權限 : 接續剛剛建立使用者的部分，切換分頁到`AdministrativeRoles`把該給的權限給一給，然後最後一個分頁`SchemaPrivileges`把剛剛開好的DB開放給他使用

以上即為mysql設定

## 3. 安裝Wordpress
> 下載頁:[Taiwan 正體中文— WordPress](https://tw.wordpress.org/releases/){: target="_blank"}

一樣，下載，解壓縮到喜歡的位置，我是直接放在`C:\inetpub\wwwroot\wordpress`這樣比較省事，不用再設定一次IIS的資料夾權限

### 設定
跟剛剛安裝PHP很類似，於Wordpress根目錄內有個`wp-config-sample.php`檔，把它複製一個改名為`wp-config.php`，他即為你正式的設定檔  
而要異動的設定如下:
+ 前幾行的DB連線資訊 : 把你剛剛在mysql設定的資料庫名稱跟連線位址`localhost:port`還有帳密通通加進去吧。
+ 認證唯一金鑰設定 : 就在DB設定的下一段，註解有詳細說明如何處理，這些不弄的話，初次開啟Wordpress安裝畫面時會顯示Table Insert失敗。
+ 最後拉到最下面新增一段`define('FS_METHOD', 'direct');` 這是讓Wordpress可以在後台直接安裝外掛程式需要的功能。

## 4. IIS設定
iis除了把站點開起來之外還有以下事情要設定:

### Default Document
新增`index.php`做為站點的預設入口
### Handler Mappings
進入後於右上角選擇`Add Module Mapping...`，然後照下圖設定吧
![](/public/res/php_cgi.jpg)
> FastCgiModule如果選不到的話，要去Windows新增功能，分類在
> 網頁伺服器(IIS) / 網頁伺服器 / 應用程式開發 / CGI

以上完成設定後就可以用IIS跑PHP應用程式了，此時即可啟用該站點  
並開啟瀏覽器前往`http://localhost/wp-admin/install.php`，順利的話就會看到歡迎畫面了

到此完成安裝。
![](/public/res/everybodygohome.jpg)

# 後續問題
### 外掛安裝失敗
於安裝完成後，遇到一個問題是每次要安裝外掛他都顯示  

    No working transports found

上網估狗了一下發現似乎是curl無法使用  
解法如下: 
1. php設定: 請回頭檢查php extension的curl跟openssl有沒有開
2. 複製lib檔: 將php中libeay32.dll,ssleay32.dll,php_curl.dll,libssh2.dll複製到windows/system32
3. 安裝[curl](https://curl.haxx.se/download.html#Win32){: target="_blank"}: 
    + 下載`Win32 CAB`
    + 解壓縮，然後把目錄加到環境變數