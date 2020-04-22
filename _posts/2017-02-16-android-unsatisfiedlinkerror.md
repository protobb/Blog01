---
layout: post
title:  "Android UnsatisfiedLinkError 問題"
---

Android的NDK是個麻煩的東西ˊ_>ˋ  
如果有的Library有編譯arm-v7a有的卻沒有編譯，或者這個module有支援x86那個卻沒有，這種情況下就非常容易出現惱人的`java.lang.UnsatisfiedLinkError`  

<!-- more -->

看到這個幾乎都是NDK的so檔有問題XD

## 解法 ##
直接在gradle寫清楚本專案只支援哪幾種ABI,作法如下：

{%highlight gradle%}
defaultConfig {
    applicationId "your.app.package"
    minSdkVersion 17
    targetSdkVersion 24
    versionCode 17
    versionName "1.0"
    ndk {
        abiFilters 'armeabi-v7a', 'x86'
    }
}
{%endhighlight%}

重點為ndk區塊，`abiFilters`可自行依照需求增減這幾種處理器

+ x86
+ armeabi-v7a
+ armeabi
+ x86_64
+ arm64-v8a