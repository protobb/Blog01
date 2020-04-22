---
layout: post
title: 在有安裝haproxy的伺服器更新SSL憑證
image: /public/res/letsencrypt.jpg
---

![](/public/res/letsencrypt.jpg)

Let's Encrypt憑證每次有效期限三個月，今天剛好需要更新，由於強者我同事沒有把他寫成排程，所以只好慢慢來研究怎麼更新了。

之前每次更新都搞得人仰馬翻的，這次終於有那個閒工夫稍微紀錄一下工作流程

<!-- more -->

## 步驟
1. 取得最高權限

        $ sudo -i

2. 關閉haproxy服務

        $ service haproxy stop

3. 執行[certbot](https://certbot.eff.org/)

        $ certbot renew

    > 這一個步驟老是在出錯，下次遇到再來看看是否真的有解決問題了 (汗

4. 把renew好的金鑰移到正確的地方

        $ cd /etc/letsencrypt/live/YOUR.SITE.NAME
        $ cat fullchain.pem privkey.pem > /etc/haproxy/certs/YOUR.SITE.NAME.pem

    > YOUR.SITE.NAME 記得替換成你們家的網域

5. 重新啟動haproxy

        $ service haproxy start

## 後記
以上的行為應該要寫成排程定時執行才對 (例如六個月過期那就每五個月執行一次之類的)  
但我合理懷疑公司之前的IT同事選擇留一手，也就是~~如果我離職了就沒有人有能力維護~~ ，那既然這樣我也從善如流，繼續這個陋習囉(?)