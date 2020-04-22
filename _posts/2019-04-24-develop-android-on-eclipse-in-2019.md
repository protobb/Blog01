---
layout: post
title: 2019年在Eclipse寫Android的指南
description: 在2019年，如果你還想用Eclipse開發Android的專案，想必你一定有著滿腹的委屈與說不完的苦衷，就跟我一樣。
image: /public/res/android-1869510_1280.jpg
---

![](/public/res/android-1869510_1280.jpg)
在2019年，如果你還想用Eclipse開發Android的專案，想必你一定有著滿腹的委屈與說不完的苦衷，就跟我一樣。

<!-- more -->

## 1.安裝Eclipse
此部分我就不再花篇幅整理了，就照步驟裝，應該不會出差錯才是。 

## 2.安裝ADT
ADT全名Android Development Toolkit，把它當成是Eclipse跟Android SDK 溝通的工具吧

1. 於主畫面工具列選擇`Help/Install New Software`
![](/public/res/eclipse-install-new-software.jpg)
2. 於彈出視窗點選`Add`添加新的套件庫
    + Name: 隨便你填，我填`Android`
    + Location: `http://dl-ssl.google.com/android/eclipse`
    ![](/public/res/eclipse-repo-android.jpg)
3. 把顯示出來的東西全部裝一裝，跳什麼警告通通給他同意就對了
4. 最後會請你指定SDK的目錄，此部分就待下個步驟完成後填上吧

## 3.安裝SDK
這步驟才是這篇文章真正要講的東西，你要是胡亂裝就會沒辦法用了。  
在整理出這篇心得之前，我曾經幹過以下的事情：
+ 跟Android Studio共用SDK
+ 去[AS官網](https://developer.android.com/studio/#downloads){: target="_blank"}下載最新的獨立SDK Tools

結果就是通通都不能用，只會顯示錯誤訊息

    Failed to get the required ADT version number from the SDK
    The Android Developer Toolkit may not work properly.

### 正確的作法
會出錯的原因是Google早就[停止支援ADT](https://android-developers.googleblog.com/2016/11/support-ended-for-eclipse-android.html){: target="_blank"}了，現在的SDK都是專為Android Studio設計的，所以你要是用最新提供的SDK必死無疑。**請獨立下載一個舊版的，並放在另一個資料夾裡面**

舊版SDK下載連結：
+ [android-sdk_r24.4.1-windows.zip](https://dl.google.com/android/android-sdk_r24.4.1-windows.zip)
+ [android-sdk_r24.4.1-macosx.zip](https://dl.google.com/android/android-sdk_r24.4.1-macosx.zip)
+ [android-sdk_r24.4.1-linux.tgz](https://dl.google.com/android/android-sdk_r24.4.1-linux.tgz)

完成後回到Eclipse，在`Windows/Preferences/Android`分頁內，指定你的資料夾位置後存檔重開
![](/public/res/your-sdk-loc.jpg)

最後在工具列的`Windows`選項內找到你的SDK Manager，一切順利的話應該就可以看到畫面，可以開始選擇你要安裝的版本了  
ヽ(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)人(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)ﾉ

## 後記
不管你最後怎麼解決了Eclipse上的問題，請永遠不要忘了，趕快把專案轉移到Android Studio上才是正確的作法。過時的東西就讓它入土為安吧(合十)