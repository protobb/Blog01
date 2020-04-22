---
layout: post
title: "在 Vue.js 拿到上傳檔案的事件"
---

原本嘗試遵照 [在網頁應用程式中使用本地檔案](https://developer.mozilla.org/zh-TW/docs/Web/API/File/Using_files_from_web_applications){: target='_blank'} 的方法用`onchange`事件去提取檔案，卻一直提取失敗。

```
<input type="file" id="input" @change="handleFiles(this.files)">
```

<!-- more -->

## 猜測

猜測可能的原因應該是那個`this`在寫成vue的語法之後遺失了，所以要想辦法繞開。

## 解法

不指定參數，讓瀏覽器自己傳事件進去


html模板：
```
<input type="file" id="input" @change="handleFiles">
```

vue：
```javascript
methods: {
  handleFiles: function (element) {
    let file = element.target.files[0];
    let reader = new FileReader();
    reader.onload = () => {
      this.convert(reader.result);
    };
    reader.readAsText(file);
  },
  convert: function(str){
    //TODO
  }
}
```