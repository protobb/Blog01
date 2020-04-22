---
layout: post
title:  "手動建立 Sitemap"
---

`Sitemap.xml` 是一個放在網站根目錄的檔案，用來記錄這個網站包含哪些網址，提供給搜尋引擎建立索引之用。

一般來說如果你使用的是主流的 CMS 系統，這東西是不用自己生成的，會有原生支援，或者是一些擴充套件可以自動幫你產生，但如果你整個網站都從頭自己寫，那 sitemap 基本上也要自己生出來。

<!-- more -->

# 方法1: 借助線上服務
你可以把你的網站上線之後，使用一些線上的服務掃描一遍，然後自動產生。  
這樣的服務通常會有一些限制，但一般情況而言應該都是堪用的，例如這個網站就有提供這樣的服務：[XML-Sitemaps.com](https://www.xml-sitemaps.com/){: target="_blank"} 

# 方法2: 自己寫程式跑
我自己的案例比較特殊，我自己持有一個網址列表，所以我就寫了一小段程式把陣列轉成 xml 格式的檔案。

## xml的格式
一個最基礎的 sitemap 應該長得像這樣：
```xml
<?xml version="1.0" encoding="utf-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
 <url>
  <loc>https://janelin612.github.io/</loc>
 </url>
 <url>
  <loc>https://janelin612.github.io/page2/</loc>
 </url>
</urlset>
```
+ 以起始 `<urlset>` 標記做為開頭，並以結束 `</urlset>` 標記做為結尾。
+ 指定 `<urlset>` 內的名稱領域 (通訊協定標準)。 
+ 讓每個 URL 中包含一個 `<url>` 項目做為母層 XML 標記。
+ 在每個 `<url>` 母層標記包含一個 `<loc>` 子層項目。

> 以上說明取自 [sitemaps.org](https://www.sitemaps.org/zh_TW/protocol.html){: target="_blank"} 

## 撰寫程式
了解 xml 規格後，就寫成 node.js 的程式吧！  
其中 json-to-xml 的程式用到了 [xml-js](https://www.npmjs.com/package/xml-js){: target="_blank"}
```cmd
$ npm i xml-js
```
完整的 js 程式碼如下：
<script src="https://gist.github.com/janelin612/0f689a91c12edf08f405044e74a1f39a.js"></script>

+ 輸出的時候帶兩個參數
    + `compact: true`：必填，這樣才會生成正確的結果
    + `spaces: "\t"`：選填，沒有帶的話會全部擠成一行，有礙觀瞻(?)

# 提交 Sitemap
寫好之後要記得提交啊

### 1.偷懶的作法
你可以直接 ping 一個網址即可快速提交  
```
https://www.google.com/ping?sitemap=YOUR_SITEMAP.xml
```
### 2.乖乖去 Search Console 註冊網站
[Google Search Console](https://search.google.com/search-console/about){: target="_blank"} 註冊了之後你未來可以來這裡看大家都用什麼搜尋關鍵字進你的網站，何樂而不為？