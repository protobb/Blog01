---
layout: post
title: Android Studio上的SVN設定
description: 紀錄我在AS上安裝SVN時做過的一些設定
---

雖然現在版本控制還在使用SVN而不是使用git已經越來越少見，但作為一個小小菜鳥實在是沒有那個能力去要求公司遷移至git，所以就還是將就配合一下吧。

<!-- more -->

## 安裝TortoiseSVN
[TortoiseSVN](https://tortoisesvn.net/)作為SVN在Windows上最受歡迎的GUI工具，基本上安裝他就對了，就算你不使用Android Studio作為IDE還是可以在檔案總管直接操作SVN功能。  
需要注意的應該只有以下兩點：

+ 安裝時記得把command line tool勾起來
![](/public/res/2019-03-12_151030.jpg)
+ 安裝好之後記得去系統把環境變數裡的`Path`加上你的資料夾，例如`C:\Program Files\TortoiseSVN\bin`

## Android Studio設定

### 執行檔設定
基本上如果上一步的環境變數有指定好，那AS裡面是無須重新指定`svn.exe`的執行檔位置，反之如果沒有設定環境變數，那們你需要在`Settings/Version Control/Subversion`內設定。

以上都搞定之後，就可以開開心心的去把你的專案checkout下來了  
確認專案可以正常建置後，可以再來完成最後一件事情。

### Ignored Files
如果有在AS上使用git的經驗，應該會注意到Android Studio會很貼心的自動幫你把`.gitignore`給生成好，但很不幸的svn並沒有這樣的好康，所以請自己設定吧。  
![](/public/res/2019-03-12_161656.jpg)  
設定的位置在`Settings/Version Control/Ignored Files`裡面，從右手邊一筆一筆添加下列內容：

    File: .DS_Store
    File: .idea/workspace.xml
    File: local.properties
    Directory: .gradle/
    Directory: .idea/libraries/
    Directory: build/
    Directory: app/build/
    Mask: *.iml 
    Mask: *.iws

這樣做可以避免自己不小心把垃圾commit出去，害你同事的環境爆炸(ﾟ∀。)
