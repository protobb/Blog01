---
layout: post
title:  "Vue.js在IE的相容性"
---

交付的網站被客戶反映不支援IE，對此我原本是不抱持任何希望，覺得IE不可能可以用Vue，大概只能跟客戶吵架，或者是全部改寫了...

但花了點時間仔細探究原因，赫然發現其實Vue是可以在IE上面運作的，只是需要注意這幾件事情：

<!-- more -->

## 1. Head宣告相容性 ##

{% highlight html%}
<meta http-equiv="X-UA-Compatible" content="ie=edge">
{% endhighlight %}

html的head區塊要宣告`X-UA-Compatible`這個meta data，告訴IE使用最新的標準進行渲染

## 2. 不要使用箭頭函數(Arrow Function) ##
此部分才是我程式掛掉的主要原因，我把它用在`vue-resource`的程式碼上，例如

{% highlight javascript%}
$http.get(API_URL).then(
	(res)=>{
		//do sometihng...
	},
	(res)=>{
		//when error...
	}
)
{% endhighlight %}

其中有用到`=>`的部分就是IE不支援的部分，要改回傳統的做法，乖乖傳一個callback進去

{% highlight js%}
$http.get(API_URL).then(
	function(res){
		//do sometihng...
	},
	function(res){
		//when error...
	}
)
{% endhighlight %}

# 後記 #

目前測試的結果改成這樣我的畫面就正常顯示了，真是可喜可賀，不然如果全部要砍掉改回用jquery重寫一次我應該會哭倒在辦公桌前。

唯一的問題就是我只測了IE11，不知道可以相容到IE幾....希望客戶不會跟我說IE6要可以用 ・゜・(PД`q｡)・゜・