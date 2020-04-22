---
layout: post
title:  "Android的程式碼模糊"
---

# 關於程式碼模糊 #
Java的運作方式是會先編譯出中間碼，然後才由JVM去轉成機器碼，藉此達到跨平台。這樣的特性導致從中間碼回推原始程式碼是可以輕易達到的，因此，為了讓拿到你程式的人沒辦法輕易解讀你的source code，你需要執行程式碼模糊。

<!-- more -->

他的基本原理是把變數名稱，方法名稱通通用無意義的字母取代，例如你原本的method長這樣：

{%highlight java%}
api.login( user.getId() , user.getPassword() );
{%endhighlight%}

執行完之後可能會變成這樣：  

{%highlight java%}
a.c( bb.a() , bb.b() );
{%endhighlight%}

# Android上的程式碼模糊工具 #
如果你是使用Android Studio開發，那執行程式碼模糊是一件很簡單的事情，你只要去`\app\build.gradle`這個檔案裡面把`minifyEnabled`設為true就完成了...  

當然，很多時候事情沒有這麼單純XD  
如同上面提到的，模糊工具會改變method的名稱，所以只要有用到JNI，或者使用java的反射去找method的程式，通通都會出錯，這時候你就會發現，原本開發中直接送進手機的debug APK都能用，但匯出Signed APK的時候就會編譯失敗。 
這問題特別容易出現在你有引入third-party library的時候，畢竟自己寫的自己會注意，別人寫的往往會忘記。  
於是，你需要一些補救措施。

# 編譯失敗怎麼辦? #

## 1.修改DefaultProguardFile文件  
在剛剛設定`minifyEnabled`的底下還有一行`proguardFiles`，用來指定Proguard的設定文件，把原始的`proguard-android.txt`改成`proguard-android-optimize.txt` (不確定該步驟是否必要)

## 2.加入例外清單
該步驟才是重點，請在`\app\proguard-rules.pro`內針對有需要保留名稱的class 或者method設定例外，並關閉警告。  
例如如果你要把android support library相關的class全部保留，就這樣寫：

	-keep class android.support.** { *; }

或者你只要保留一個method：

	-keep class com.example.MyClass {
	  public void myMethod(int);
	}

然後是把某個class關閉警告：

	-dontwarn com.example.MyClass.**

至於有哪些需要被保留，就要自行參考編譯失敗的錯誤訊息了XD

# 其他用法 #

1.停用Logcat：輸出正式apk的時候，往往需要把所有debug用的log拿掉，除了慢慢註解之外，可以直接用Proguard挖掉：

	-assumenosideeffects class android.util.Log {
	    public static *** d(...);
	    public static *** v(...);
	    public static *** i(...);
	    public static *** w(...);
	    public static *** e(...);
	}

2.縮小檔案：關於Proguard這個工具，除了可以模糊之外，他還會把你整個專案內沒有用到的class，以及方法通通過濾掉，連編譯都不編譯，再加上混淆後的變數名稱通常會變短，於是結果就是最後的apk檔會變小。

# 缺點 #
執行程式碼模糊有個顯而易見的缺點，就是程式當機的時候，你收到的Call Stack也是模糊後的結果，所以你也看不懂(汗)  
推薦使用[Firebase](https://firebase.google.com/)的Crash Report功能，他可以上傳mapping檔，把有沒有模糊的兩種結果對照起來，有相關需求的人可以考慮採用。