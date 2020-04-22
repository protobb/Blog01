---
layout: post
title: PHP5 與 MySQL8 的相容性問題
---

之前為了調整 Wordpress 版型問題  
嘗試將 php7 降版到 php5，結果卻遇到了奇怪的問題...
```
Server sent charset (255) unknown to the client. Please, report to the developers in...
```

<!-- more -->

## 原因
上網估狗了一下，其實答案蠻好找的  
原因是因為 MySQL 在8.0之後更改了預設的 charset  
從原本的`utf8`改成`utf8mb4`  
結果php5就不認得啦

## 解法
既然知道了問題，那做法便是把預設的 charset 改回`utf8`就好了。  
網路上看到都是要去 configuration file 改設定 
```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
character-set-server = utf8
```
結果我卻卡在這裡...   
因為我用wizard裝起來的，所以從頭到尾都沒有改過設定檔  
於是我的問題變成了:  
**Windows版的MySQL ConfigurationFile究竟在哪裡????**

## 尋找設定檔
跑去stackoverflow爬文，得到了一堆奇奇怪怪的答案  
每個人都提供了不一樣的位址，然後有的人的設定檔叫做`my.ini`，有的叫做`my.cfg` (你們搞得我好亂啊)

最後記錄一下真正讓我找到檔案位址的作法：

1. `WIN`+`R` 執行`services.msc`
2. 找到你的MySQL服務，如果照預設的會叫做MySQL+幾個數字 (我的叫做MySQL80)
3. 打開查看細節，會看到執行檔的位置 "C:\Program Files\MySQL..." 
4. 上面那串字串後面會帶個參數`--defaults-file=` 參數後的檔案就是你的設定檔啦!

## 後記
結果問題解決了，買來的版型還是不能用 ｡ﾟ(ﾟ´ω`ﾟ)ﾟ｡  
最後設計師還是決定換一個版型，而我也換回使用php7

有種白忙一場的感覺呢。
