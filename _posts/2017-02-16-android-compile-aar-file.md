---
layout: post
title:  "Android 編譯AAR檔"
---

## 前言 ##
基於某些理由，例如：

+ 拆出去方便管理
+ 有些小tools想要每個自己寫的專案都可以使用，卻嫌複製貼上太愚蠢
+ 你想要保護某些部分的程式，不想交出原始碼

這時候你可能需要將一個原始的專案包裝起來之後以aar檔的形式讓其他專案可以引用。

<!-- more -->

## 作法 ##

1. Gradle
	1. 將`module.gradle`的第一行從`apply plugin: 'com.android.application'`後面的部分改成`'com.android.library'`
	2. 移除下方`defaultConfig.applicationId`，library 型態的專案不允許有這個欄位	
2. AndroidManifest.xml
	1. 把跟啟動有關的`<intent-filter>`清空，通常會在MainActivity的區塊裡面
	2. 把application區塊的`android:icon`和`android:label`移除，不然會跟引用他的專案相衝
3. 重新Build一次專案吧
4. 出來的檔案會在`.\app\build\outputs\aar\`裡面

## 如何引用 ##
1. 將剛剛build好的aar檔丟到`.\app\lib\`裡面
2. 編輯`moudle.gradle`
	1. `android`區塊新增以下內容
	
			 repositories {
		        flatDir {
		            dirs 'libs'
		        }
		    }
	2. `dependencies`區塊新增以下內容
	
			compile(name: 'yourAarFileName', ext: 'aar')
3. Sync your gradle~ 

## 注意 ##

- aar檔自己的gradle引用的所有dependencies，是不會被編譯進去的，所以都需要在引用AAR的專案再import一次
- 如果有啟用程式碼模糊，記得將public方法取消，不然沒人能用你的套件 : \ 