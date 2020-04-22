---
layout: post
title:  "Tomcat Servlet 設定同源政策"
---

由於本次交付客戶的程式被要求要設定相關的參數，因此稍微研究了一下同源政策(Same-origin policy)是啥，並記錄一下怎麼在tomcat伺服器上完成設定。

<!-- more -->

# 1. 同源政策 (Same-origin Policy)

## 簡介
[MDN上關於同源政策的文件](https://developer.mozilla.org/zh-TW/docs/Web/Security/Same-origin_policy){: target="_blank"}  
同源政策指的是定義網頁上的程式碼可以使用那些網域的資料，包括.js檔，圖片資源，以及CSS文件都會被限制。  

很容易理解的，同源政策的存在是為了防止跨網域攻擊XSS。簡單來說就是怕你不小心載入了別人的攻擊程式碼，乾脆限制只有我指定的某些網域的程式碼可以被執行 :D

## 設定方式
基本上設定方式都是在header加東西。由於我只被客戶要求要設定`Content-Security-Policy`，所以以下也只介紹這個XD

直接用我最後設定好的CSP當成範例吧：

    Content-Security-Policy : default-src 'self';script-src 'self' https://cdnjs.cloudflare.com 'unsafe-inline' 'unsafe-eval';style-src 'self' 'unsafe-inline' ;img-src *

	
總共有這幾種資源可以設定

- `defalut-src`	：如果以下有東西沒設定就會讀取這個
- `script-src`	：關於javascript的部分
- `style-src `	：關於css的部分
- `img-src` 		：關於圖片的部分

然後有這幾種設定值

- `*` ：很好懂，全部放行的意思
- `'self'` ：只有跟自己同網域的資源才放行
- `http://xxxx` :  指定某個網域放行，如我的範例就對CDN過來的js檔放行
- `none`：全部禁止

除上述之外，CSP還另外定義了一些跟javascript還有CSS有關的設定值

- `unsafe-inline` ： 若你在`script-src`跟`style-src` **沒有**宣告這個選項，則你的該網頁預設會禁止使用inline程式碼，也就是以下這種寫法會無法執行

{% highlight html %}
<script type="text/javacript">
	console.log("這段無法執行~")
</script> 
{% endhighlight %}

只能從外部引入js檔
		
{% highlight html %}
<script src="./your_code.js"></script>
{% endhighlight %}		

CSS檔同樣道理。
	
- `unsafe-eval` ： `eval()`是個超強的function，所以也很危險。 與上面的一樣，若你**沒有**在`script-src`宣告這個選項，則你的網頁會直接禁用`eval()`


綜合以上，重新檢視一下我的設定值

	Content-Security-Policy : 
		default-src 'self';
		script-src 'self' https://cdnjs.cloudflare.com 'unsafe-inline' 'unsafe-eval';
		style-src 'self' 'unsafe-inline' ;
		img-src *

- 預設只有同網域的資源可以載入
- script的部分允許同網域、指定的CDN網址可以載入，並允許inline-script & eval函式
- css的部分只允許同網域，並允許inline-script
- 圖片的部分全部放行

## 備註

- 不同資源用 `;` 隔開
- 保留字要上單引號
- 網址不用加單引號
- 同一個資源的多個設定值用空格分開

# 2. 在Tomcat Servlet 設定 header

簡單說明:

1. 實作一個`javax.servlet.Filter`
2. 在他的`doFilter`方法內對`ServletResponse`加料
3. 在全域的`web.xml`檔宣告整個網站的子目錄皆適用此`Filter`

## 2-1 實作Filter

{%highlight java%}
public class MyFilter implements Filter {
  @Override
  public void doFilter(ServletRequest req, ServletResponse resp,
      FilterChain chain) throws IOException, ServletException {
    
    HttpServletRequest httpReq=(HttpServletRequest)req;
    HttpServletResponse httpResp=(HttpServletResponse)resp;
    httpResp.addHeader("Content-Security-Policy", "YOUR_CONFIGURE");
    chain.doFilter(httpReq, httpResp);
  }
}
{%endhighlight%}

## 2-2 在`web.xml`宣告適用範圍

{%highlight xml%}
<filter>
    <filter-name>CSP-Filter</filter-name>
    <filter-class>YOUR.PACKAGE.YOUR_FILTER</filter-class>
</filter>
<filter-mapping>
    <filter-name>CSP-Filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
{%endhighlight%}

以上。