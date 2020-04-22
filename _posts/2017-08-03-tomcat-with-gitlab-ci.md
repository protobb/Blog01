---
layout: post
title:  "Tomcat + Gitlab CI 配置持續整合環境"
image: /public/res/WAzZmtT.jpg
---

![](/public/res/WAzZmtT.jpg)

自從公司 gitlab 架起來之後，一直想要來試試看有沒有辦法實現自動部署 j2ee 的 web application 到客戶家的 tomcat server 上。  
經過一整個下午的奮鬥，於是有了這篇簡單的紀錄。

<!-- more -->

# 環境 #

- GitLab Community Edition 9.3.4
- jre 1.6
- tomcat 6.0.35

# 第一部分：從 CMD 編譯 WAR 檔

一直以來這個專案都是用 Eclipse 開發，所以要生成 war 檔也是直接從 Eclipse 中對專案按右鍵執行匯出的方式處理，因此想要整進 CI 第一個要搞定的就是這個。  

## 1.編譯純Java檔的部分 ##

專案除了 jsp 的檔案之外，還有一些 `.java` 檔需要先編譯好，再一起包進去。此部分就是很單純的 Java 編譯，指令如下：

{% highlight shell %}
cd ./src
find -name "*.java" > sources.txt
javac @sources.txt  -encoding UTF-8 -cp "Test.jar:../WebContent/WEB-INF/lib/*" -d "../WebContent/WEB-INF/classes"
rm sources.txt
{% endhighlight %}

大致上的意思就是切進 `./src` 的資料夾底下，然後把所有副檔名是 `.java` 的檔案整理成一份清單，最後照這份清單依序編譯。  

- `-encoding` 用來指定編碼，我用的是 UTF-8
- `-cp` 用來指定所有有用到的 `.jar` 檔所在的位置
- `-d` 用來指定編好的 `.class` 檔要放在哪個資料夾

## 2.將整個專案封裝成WAR檔 ##

上網估狗了各種資料，最後整理出來的 command line 指令如下:

{% highlight shell %}
cd ./WebContent
jar -cvf {PROJECT_NAME}.war * WEB-INF
{% endhighlight %}

也就是先進到專案根目錄的下一層`/WebContent`內，其他的 Eclipse 相關的東西通通不編譯進去，然後才下包裝的指令。參數的部分稍微解釋一下
	
 - `-v` 是用來將細節print出來，可下可不下，`-c` 跟 `-f` 則是一定要的東西
 - 第二個是war檔的檔名，自己決定吧
 - `*` 意味著整個目錄 (`./WebContent/*`) 內的東西通通都要進去
 - `WEB-INF` 則是自己指定自己的 manifest 檔案所在目錄 (`./WebContent/WEB-INF`) ，這部分由於 Eclipse 幫我建好了，所以我如果沒有指定的話，部署上去之後會無法啟動  

執行完後應該可以在目錄看到你的 war 檔了，可以拿去部署看看能否成功 ：D

# 第二部分：Tomcat 環境準備 #

tomcat 需要準備的部分只有開放透過 http request 就可以完成 undeploy & deploy 的功能即可。

1. 編輯使用者權限：透過 vim 開啟設定檔 `TOMCAT_ROOT\conf\tomcat-user.xml` 改成如下面所示

{% highlight xml %}
<?xml version='1.0' encoding='utf-8'?>
<tomcat-users>
  <role rolename="manager"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <user username="USER_NAME" password="USER_PW" roles="manager-gui,manager-script"/>
</tomcat-users>
{% endhighlight %}

就是新增一個 rolename 叫做 manager-script ，然後指定給你的 user

2. 存檔，然後重啟 tomcat

完成設定後即可以使用此兩個網址進行 undeploy & deploy 的動作：
	
	http://{USER}:{PW}@{SERVER_URL}:{PORT}/manager/undeploy?path=/{WAR_PATH}
	http://{USER}:{PW}@{SERVER_URL}:{PORT}/manager/deploy?path=/{WAR_PATH}&update=true

# 第三部分：gitlab-ci

其實同事已經幫我做掉這一塊最麻煩的事情，也就是準備一個 Runner 。  
因此這個部分我也沒辦法紀錄些什麼，只知道東西似乎是用 Docker 架起來的XD 

在 Runner 已經準備好的情況，唯一需要做的事情就是在專案根目錄新增`.gitlab-ci.yml`檔即可，我寫出來的檔案如下

{% highlight yml %}
image: java:6

stages:
  - build
 
build:
 stage: build
 tags: 
   - linux
 only:
   - master
 script:
   - cd ./src
   - find -name "*.java" > sources.txt
   - javac @sources.txt  -encoding UTF-8 -cp "Test.jar:../WebContent/WEB-INF/lib/*" -d "../WebContent/WEB-INF/classes"
   - rm sources.txt
   - cd ../WebContent
   - jar -cvf {PROJECT_NAME}.war * WEB-INF
   - curl "UNDEPLOY_URL_AS_ABOVE"
   - curl --upload-file ./{PROJECT_NAME}.war "DEPLOY_URL_AS_ABOVE"
{% endhighlight %}

一樣稍微解釋一下我覺得需要說明的部分

 - `image` 的部分指定了要用 java 1.6 進行編譯
 -  `tags` 我上了`linux`字樣，此部分要跟 Runner 配合。因為我們公司備了兩台 Runner ，一台是 Windows 一台是 Linux ，所以我需要指定強制用帶有`linux`這個tag的那台 Runner 編譯
 -  `only master` 應該還算好懂，只有master有動作時會觸發
 -  剩下的 `script` 指令就理解成會在 cmd 執行的東西就可以了

以上，寫完之後 `git push master` 然後就可以去 gitlab 網站上看執行的結果了。


# 後記

1. 其實我猜 Runner 才是最難搞的部分，結果我剛好可以撿同事現成的東西來用，省下了不少工啊 XD
2. 改天再來把 slack 的 webhook 掛上去，未來可以直接從 slack 上面看到部署結果 