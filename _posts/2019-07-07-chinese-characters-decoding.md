---
layout: post
title:  "JS處理中文亂碼問題"
image: /public/res/hex-to-string.jpg
description: "記錄一次中文亂碼的轉檔過程並分享最後的解法"
---

昨天從FB下載了一份包含所有對話紀錄的JSON檔，但裡面的中文字編碼有問題，通通變成下面這個樣子
```json
"\u00e7\u0094\u009f\u00e6\u0097\u00a5"
```
本來想說這種事情應該很簡單，就把他每個字元當成`char`轉存不就得了，結果轉出來的東西從網頁上看還是一團亂碼...

<!-- more -->

## 錯誤的嘗試
既然轉換之後仍然是亂碼，那至少代表這個編碼不是`utf-8`，那會不會是`big5`或其他現在比較少看到的編碼呢？所以我試著切換 html 檔的 charset，但仍然得到錯誤的結果
```
çæ¥å¿«æ¨å  //這到底是什麼鬼東西
```

## 整理思路
後來注意到一件小事情是，他所有的16進位資料都是`\u00`開頭，或許意味著只有後兩位有意義？於是我把字頭全部取代後得到下面的結果
```
E7 94 9F E6 97 A5
```
這東西看起來很像是在使用記憶體修改器會看到的東西，於是我稍微有思緒了：他實際上應該是`byte[]`。也循線找到了網站測試證明了我的想法是正確的

![](/public/res/hex-to-string.jpg)
> [Hex decoder: Online hexadecimal to text converter](https://cryptii.com/pipes/hex-decoder){: target="_blank"}

## byte array to string
搞懂他到底是什麼編碼格式之後，就是要開始寫轉檔程式了（總不能一行一行貼上去面的網站翻譯吧）在 stackoverflow 上有這麼一篇提問：[How to convert UTF8 string to byte array?](https://stackoverflow.com/a/18729931){: target="_blank"} 下方的解答有人分享了轉換的規則：

+ 多位元組的情形下，第一個位元組前面`1`的數量即代表該字元所需要的位元組數量。扣除這個資訊所占用的位元後剩下的位元即為資料區塊
+ 後面接著的延續位元組，其格式固定為`10`字頭，後面的六個位元為資料區塊
+ 將所有的資料區塊全部拼起來即為`utf-8`碼

以我上面分享的第一個字為例子   
前三碼`E7 94 9F`轉成二進制的資料呈現以下的樣子
```
11100111 10010100 10011111
```
1. 第一組前面有3個1，代表這個字要用到三個位元組
2. 扣除記錄這個資訊的區塊後，剩下的`00111`即為第一塊資料區
3. 第二組為延續位元組，其前兩碼固定為`10`，後面的`010100`即為資料區
4. 第三組為延續位元組，其前兩碼固定為`10`，後面的`011111`即為資料區
5. 三組資料全部拼起來`00111010100011111`就是這個字的 utf-8 碼
6. 把這個二進制資料轉回 int 後再轉成 char 就會看到正確的中文字了
```javascript
String.fromCharCode(parseInt('00111010100011111',2)) //'生'
```

## 寫成程式
最後就是把上面的邏輯寫成程式碼了  
<script src="https://gist.github.com/janelin612/464328333f11b9332a3dbadf38c3dfc6.js"></script>

## 後記
+ 好久沒有碰到需要直接操作位元的問題，大學修的計概都還給教授了Orz
+ 組資料的地方如果可以直接用`<<`跟`>>`進行位元計算性能應該會比字串處理提升很多，改天再來修改程式碼吧。
+ 實測發現emoji轉不出來🤔🤔🤔

---

## 補充
整篇文章完成後才發現前面寫了一大堆，其實有更簡單的做法  
```javascript
function decode(text) {
    if (!text)
        return "";

    let charArr = [];
    for (let i = 0; i < text.length; i++) {
        charArr.push(text.charCodeAt(i));
    }

    return new TextDecoder().decode(new Uint8Array(charArr));
}
```