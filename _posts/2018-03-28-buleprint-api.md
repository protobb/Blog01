---
layout: post
title:  "使用Blueprint撰寫API文件"
image: /public/res/blueprintapi.jpg
---
![](/public/res/blueprintapi.jpg)

稍微記錄一下使用[Blueprint](https://apiblueprint.org/){: target="_blank"}作為API文件生成工具的小小心得
    
我自己過去都用Markdown在寫文件，但是基於md檔本身其實格式過於基本，對於要排版出好閱讀的API文件依舊稍嫌不足，Blueprint本身的概念剛好補足了我對這塊的需求：

> 撰寫一個md檔，然後輸出一個漂亮的API文件

<!-- more -->

## 前置作業

+ Markdown Editor : 不用多作解釋了吧，自己愛用啥就用啥，反正他就只是要寫MD檔，我目前是用VSCode
+ Aglio : 主要的渲染器，由`npm`安裝吧

```shell		
npm install -g aglio
```

## 語法介紹

[官方說明文件](https://apiblueprint.org/documentation/tutorial.html){: target="_blank"} 要學會怎麼寫文件，首先當然是看寫文件的工具的文件(這啥繞口令)    
以下簡單紀錄一下我有用到的入門語法：

### metadata

	FORMAT:1A 
	HOST: http://your.domain.com/api

寫在md檔最開頭，分別用來記錄使用的Blueprint語法版本與API的主機位址  
後者要寫不寫隨便，有寫的話Gen出來的API說明會包含完整網址

### 標題

	# 我的API文件

整個MD檔第一個 `#` 會被當成整份文件的標題

### 群組

	# Group 訂單
	
使用 `Group` 關鍵字的 `#` 會被視為API群組名稱，可以用此為API分類

### Resource

	## 訂單狀態 [/OrderStatus]
	### 查詢訂單狀態 [GET]
	使用本API可以查詢到訂單的狀態

使用 `## 說明 [url name]` 可以定義一個API的進入點(網址)，而使用 `### 說明 [method]` 則會定義允許使用的HttpMethod。 下方可以緊接著寫下詳細的API說明，一樣可以使用 `+` 定義條列式的說明

### Response Body

	+ Response 200 (applicaiton/json)
	
		{success:true,message:""}

使用`+ Response 狀態碼 (Content-type)`來定義回應 需要注意的是**如果定義了方法卻沒有定義任何回應會編譯失敗**  
另外也可以定義Header

	+ Response 200 (applicaiton/json)
		+ Header
			Key : value
		+ Body
			{success:true,message:""}

### Request Body

	+ Request (applicaiton/json)

		{var1:"0",var2:"1"}

與 `Response Body` 定義起來的感覺很像

### Url Parameter
這個比較複雜一點，如果你的API是使用Http Get，那你的API在使用的時候像這樣：

	http://your.domain.com/api/getData?id=12345 //常見寫法
	http://your.domain.com/api/data/12345       //可以 這很restful

那你的API文件撰寫的時候要寫成這樣

	## 資料 [/getData{?id}] or [/data/{id}]
	### 取得資料 [GET]
	+ Parameter
		+ id : `12345` (integer,required) - 資料ID

+ 網址的部分要用`{}`格式化，其中如果是用`?`帶參數的要特別寫成`{?var1,var2}`
+ 底下要帶一個關鍵字為`Parameter`的列表
	+ 子列表格式意義為
	 
			參數名稱 : 範例值 (資料型態, required || optional ) - 說明
	+ 只有參數名稱必填，其他欄位都是選填
	+ **如果Parameter內的參數與網址上沒有定義的格式化字串會編譯失敗**


## 輸出
寫完後使用aglio渲染並輸出

```shell		
aglio -i MD檔名.md -o HTML檔名.html                  //最精簡的寫法
aglio --theme slate -i MD檔名.md -o HTML檔名.html    //輸出深色版本
```

## 補充
另一個我覺得寫得很好的教學文件: [https://yami.io/api-blueprint-tutorial/](https://yami.io/api-blueprint-tutorial/){: target="_blank"}