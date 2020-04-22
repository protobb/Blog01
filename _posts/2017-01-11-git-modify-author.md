---
layout: post
title:  "Git 大量修改作者資料"
---

# 前言 #
我早期的commit沒有設定好author name &email導致那幾筆commit在Github上看就是跟別人不一樣，強迫症如我，就想說找找有沒有辦法改...

<!-- more -->

# 方法 #
其實Github早就幫大家準備好Script了，照著做就可以完成  
連結於此:
[Changing author info](https://help.github.com/articles/changing-author-info/#platform-windows "Changing author info")

大致上來說就是

1. 把你的專案clone下來
2. script 複製到純文字檔內，把變數改一改
3. 存成.sh檔直接在目錄底下執行
4. 噹啷~檢視一下狀況，沒問題就force push回去

# 心得 #

雖然很簡單，但還是幾件事情需要注意：

- 實測的結果，似乎不會影響原本的樹狀結構，我超怕他把我舊的東西rebase成一條XD
- **記得remote上面所有的branch都要checkout下來**，不然執行完之後那個branch之前的節點會通通變成兩份。我就幹了這蠢事，所以看分支圖的時候在那邊納悶為啥我執行完之後有兩條master...。後來直接就把那幾條不在local端的branch砍了 :(
- push的指令Github文件上面是寫

{% highlight shell %}
git push --force --tags origin 'refs/heads/*'
{%endhighlight%}

但這樣多個bransh的時候就得一條一條push，所以我直接改成這樣：

{% highlight shell %}
git push --all --force
{%endhighlight%}