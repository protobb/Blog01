---
layout: post
title:  "Firefox只清除某些指定的瀏覽紀錄"
image: /public/res/1280px-Firefox.png
---
![](/public/res/1280px-Firefox.png)  
大概是好幾年前，那時還沒有強制禁用[NPAPI](https://zh.wikipedia.org/wiki/NPAPI){: target="_blank"}，我曾經有裝個擴充套件是可以指定刪除幾天前(例如180天前)然後瀏覽次數低於幾次的紀錄清掃器。後來隨著禁用舊式擴充套件之後這個套件也因為沒再更新而消失了。

所以我就想說那我自己進DB下SQL總可以吧？

<!-- more -->

# 資料庫在哪?
首先是路徑。以Windows版來說，Firefox的個人設定檔通常會在下列所示的位置：

    C:/Users/YOUR_NAME/AppData/Roaming/Mozilla/Firefox/Profiles/xxx.xxx

其中`YOUR_NAME`指向你的個人目錄，而後面的`xxx.xxx`是一串亂碼。   
進到目錄內後，`./places.sqlite`就是你的瀏覽紀錄與書籤的資料庫了

# 開始下SQL
## 操作工具
由於該資料庫使用的是sqlite，故我選擇在npm上安裝[`sqlite3`](https://www.npmjs.com/package/sqlite3){: target="_blank"}作為我的讀寫工具

```shell
$ npm i sqlite3
```

## 語法

首先是核心的SQL語法 
```sql
DELETE FROM moz_places 
WHERE visit_count <=2 AND 
last_visit_date < CAST(strftime('%s', datetime('2019-03-01')) AS INT)*1000000
```
+ `visit_count`是瀏覽次數
+ `datetime`函式傳入一個`YYYY-MM-DD`格式的日期，該段where語法會過濾出該日期之前的所有紀錄

完整的程式碼如下：
<script src="https://gist.github.com/janelin612/3ba9a749a14f4e07f2ec830544c71762.js"></script>

在node.js環境下執行即會清除指定的紀錄了

## 小提醒
1. 記得先備份書籤：我曾經執行後發現書籤有些東西被我刪掉了，還好我執行前有備份過，等我找到原因再來修正吧
2. 執行前可以先把`DELETE`改成`SELECT count(1)`，這樣會印出總共有幾筆資料，方便粗略評估是否有下錯條件

# 後記
當然，你可以打開你的瀏覽紀錄一筆一筆刪除，但這樣的問題是十分沒有效率。  
我個人使用上的感覺是如果你大概手動選擇個300筆左右的紀錄按刪除，火狐就會卡死沒有回應一陣子了，猜測是執行刪除指令執行了300次，產生N+1 Query問題  
> 補充: [What is the “N+1 selects problem” in ORM](https://stackoverflow.com/a/97253){: target="_blank"}

所以進瀏覽紀錄慢慢篩選慢慢刪肯定不是好辦法。  
但如果真要討論好方法，或許把這段程式碼改寫成擴充套件才是真正的好方法吧。