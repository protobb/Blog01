---
layout: post
title: 在Jekyll內對內文做簡單的客製化
description: 使用自定義的class加上自己寫的sass檔對部落格內文套用效果
---

[Jekyll](https://jekyllrb.com/) 雖然是個簡單好用的部落格工具，但是由 markdown 生成的內文，其客製化程度終究有限，所以最近把歪腦筋動到了怎麼對某個段落上一個自定義的 class 然後藉此套用自己寫的 CSS 效果

<!-- more -->

# 導入Sass檔
此部分Jekyll的[官方文件](https://jekyllrb.com/docs/assets/)就有足夠的資訊了，在此做個簡單的整理。

1. 於根目錄新增一個資料夾`./css`，並於內部放入主要的sass檔  
例如`./css/main.scss`
2. 該sass檔檔頭需要包含兩行`---`字樣，這樣Jekyll才會將其轉譯為css檔
    ```md
    ---
    # 例如這樣
    ---

    .your-class{
        width:100%;
    }

    ```

3. 於layout的header內加入同名的css檔
    ```html
    <link rel="stylesheet" href="/css/main.css">
    ```

### 使用到`@import`
其實到以上就可以開始寫scss檔了，但有一個問題  
由於開頭包含了jekyll要求的特殊語法，導致整個檔案在vscode編輯的時候會被提示語法有問題，甚至造成後面的自動完成沒辦法正常使用。  
於是我想到的作法是`main.scss`只用來引入別隻檔案，主要語法都寫在那個新檔案裡，藉此規避問題。

根據官方文件，要導入partials scss需要注意以下幾件事情：

+ 於你的`_config.yml`宣告sass檔位置，例如放在`./_sass`
```
sass:
    sass_dir: _sass
```
+ 於該目錄內的scss檔，檔頭**不用**再加`---`符號。
+ 宣告時不包含資料夾名稱。假設你的新sass檔叫做`./_sass/yee.scss`，那你在導入的時候只要寫成下列這樣即可
    ```md
    ---
    ---

    @import "yee";
    ```

# 於Markdown使用到自定義的class
上面費盡千辛萬苦寫好了css，可以使用以下方法進行宣告

### 1.直接於行內寫html
這其實是最直覺的作法，因為md檔本來就可以跟html混著寫，只是有點不美觀而已
```markdown
# 範例標題

<span class='your-class'> 
這樣就會生效了!
</span>

```
### 2.使用語法
可以使用`{: }` 這個語法對某個段落做客製化，這個方法我覺得比較好，但使用上有其侷限性就是，可能要多方嘗試一下才有辦法在正確的區塊套用效果
```markdown
+ 這行會套用效果 {: .your-class .another-class}
+ 這行又不會套用效果了
```
