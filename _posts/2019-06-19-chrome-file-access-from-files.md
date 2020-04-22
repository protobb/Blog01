---
layout: post
title:  "處理Chrome無法讀取本地端文件問題"
---

如果你有在本機的html檔內嘗試使用ajax或任何其他手法開啟另一個本地端檔案(像是載入一個json檔)，那你是無法如願使用的，此時在開發者工具會看到類似這樣的錯誤訊息：

    URL scheme must be "http" or "https" for CORS request.

因為Google Chrome預設是禁止這樣的行為的(`file-access-from-files`)

<!-- more -->

## 解法1 : 強制chrome允許file-access-from-files
參考自[這個網站](https://cmatskas.com/interacting-with-local-data-files-using-chrome/){: target="_blank"}

1. 關閉所有chrome的應用程式
2. 直接從CMD下指令
    ```shell
    $ chrome.exe --allow-file-access-from-file 
    ```

這樣可以讓chrome暫時允許讀取本機檔案，但缺點就是每次都要執行一次，而且瀏覽器關掉就失效了，很不方便。

## 解法2 : 架一個簡單的本地端server
這算是一個比較正規的解法。  
架伺服器的工具很多，如果你一時沒有什麼想法只想快速跑一個起來，那我推薦使用[npm http-server](https://www.npmjs.com/package/http-server){: target="_blank"}，他只需要兩行指令，一行安裝，一行執行：

```shell
$ npm install http-server -g
$ http-server ./YOUR_FOLDER
```
就可以在`http://127.0.0.1:8080`看到你的網站了

> 不想用node.js? 這裡有一份推薦清單供你選擇：[imgarylai/awesome-webservers
](https://github.com/imgarylai/awesome-webservers){: target="_blank"}

## 解法3 : 改用Firefox
是的你沒有看錯，直接改用火狐就可以了ヽ(✿ﾟ▽ﾟ)ノ
