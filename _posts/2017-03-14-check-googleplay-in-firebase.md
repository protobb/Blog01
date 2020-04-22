---
layout: post
title:  "Firebase檢查GooglePlayService是否可用"
---

## 前言 ##
Firebase 自從被估狗買走之後，就算是 GooglePlayService 的延伸吧，所以一樣會有如果手機沒有 GooglePlay 服務(像是中國白牌手機)就沒有 Firebase 可以用的問題。  

Firebase 本身的 API 卻沒有提供檢查該手機有沒有 Firebase 可以用的方法。還好，前面提到他一樣是基於 GooglePlayService ，所以我們可以用該手機有沒有 GooglePlayService 的結果作為 Firebase 可不可以用的依據。  

<!-- more -->

## 問題 ##
但事情總是沒有那麼單純，不然我也不用特地記下來了。  
翻了翻[官方文件](https://developers.google.com/android/reference/com/google/android/gms/common/GoogleApiAvailability)，得知我要 call 的 method叫做：

{% highlight java%}
GoogleApiAvailability
	.getInstance()
	.isGooglePlayServicesAvailable();
{%endhighlight%}

但是我在 Android Studio 呼叫時卻說找不到這個 Class...  
我一開始朝向我沒有引入GooglePlayService的 library 這個方向去解，但後來 import 進來之後卻遇到衝突，這意味著 Firebase 應該已經幫我引入好了才對。  
最後從 Package Name 開始一個一個追蹤下去才找到問題所在：

## 解答 ##

![](https://1.bp.blogspot.com/-ALHiv7mocYI/WMfjdSJwp2I/AAAAAAAAnU8/c7qFtO77_084o2rx7UwAqJZNgAW39Jt7ACLcB/s1600/2017-03-14_160552.png)
Google 不知道為啥，把 GooglePlayService 裡的幾個 public Class 執行了程式碼模糊，害我找了一兩個小時，才把這問題搞定...

希望下一版 GooglePlayService 可以把這個鳥問題修正。