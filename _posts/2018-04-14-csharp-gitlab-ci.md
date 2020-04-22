---
layout: post
title: C#.NET專案使用Gitlab CI部署至IIS
image: /public/res/WAzZmtT.jpg
---

![](/public/res/WAzZmtT.jpg)
自上次成功使用GitlabCI部署WAR檔之後，什麼專案都想嘗試用CI部署看看

<!-- more -->

## 安裝
### 1.前置準備
首先，你要有一台 IIS Server
### 2.安裝 gitlab runner
讀一下官方文件吧: [ Install GitLab Runner on Windows ](https://docs.gitlab.com/runner/install/windows.html)

簡而言之就是:
1. 把下載好的執行檔放在Server內的`C:\GitLab-Runner\`
2. cd進去執行 `gitlab-runner.exe register` 向你的Gitlab伺服器完成註冊
    - 他會問你token，那個在gitlab後台有寫
    - 然後會要你選executor，我記得我是選`Shell`
3. 執行 `gitlab-runner.exe install` 完成安裝
4. 執行 `gitlab-runner.exe start` 啟動Runner

以上，完成後可以去Gitlab後台Runner管理的畫面，確認一下是否亮綠燈，運作正常

### 3.安裝msbuild
MSBuild就是主要的建置工具，一般來說如果你有裝Visual Studio，他就會內建在裡面，但你現在要在遠端用CI建置，所以你需要額外自己安裝。

> MSBuild下載頁: [Download Microsoft Build Tools 2015 ](https://www.microsoft.com/en-us/download/details.aspx?id=48159)

### 4.安裝Nuget
Nuget是C#.NET專案使用的套件管理工具，像是`npm`那類的東西。他不包含在msbuild裡面，要自己另外裝

> Nuget下載頁: [NuGet Gallery - Downloads](https://www.nuget.org/downloads)

抓下來是一個.exe檔，找個地方放就好，我是放在`C:\`

## yml腳本
語法就兩條:
{%highlight yml%}
build:
 stage: build
 script:
  - 'C:\\nuget.exe restore'
  - 'msbuild /m /p:Configuration=Release'
{%endhighlight%}
分別會從Nuget還原套件，然後使用msbuild建置

最後去`C:\GitLab-Runner\`資料夾裡面找，會有建置完成的資料夾，此時再回去IIS設定裡面把目錄指向該資料夾，即可完成。