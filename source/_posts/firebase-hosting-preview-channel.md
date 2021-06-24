---
title: Firebase Hosting Preview Channel建立測試環境的新方法
date: 2021-04-28 21:48:29
tags:
- firebase hosting
categories:
- firebase
---
以前開發前端頁面的時候，都會建兩個hosting，一個是正式環境、一個是測試環境像下面這樣。

但現在不用這麼麻煩了，只需要建立一個hosting，至於想要在同一個hosting下產出多少個測試環境都沒關係都不會影響到正式環境，因為他會依據我們建立的每個測試環境再產出一個全新的網址出來。

會發現這個是因為最近在幫客戶改功能，但我們PM把正式環境網址跟測試環境網址都提供給客戶了，所以需要再建一個hosting作為真正的內部開發環境來用，可是還需要改個`firebase.json`和`.firebaserc`，因此也在無意間發現了Firebase Hosting Preview Channel這個不曾用過的功能。

![firebase_hosting](/firebase-hosting-preview-channel/firebase_hosting.png)

## Firebase Hosting Preview Channel

[詳細官方文件](https://firebase.google.com/docs/hosting/manage-hosting-resources)可以看這，以前我們直接deploy上去hosting的時候，都是在Live Channel上，但如下圖現在我們可以deploy很多個Preview channel，每個Preview channel都會有獨立的網址可以用，甚至可以限制該網址的有效期限。

![firebase_hosting_preview_channel_structure](/firebase-hosting-preview-channel/firebase_hosting_preview_channel_structure.png)

## 下channel指令部署到特定的hosting之下

```jsx
firebase hosting:channel:deploy --only TARGET_NAME CHANNEL_ID --expires 1d
```

- TARGET_NAME: 看你原本部署在你要的hosting都是下什麼TARGET_NAME，那就採用TARGET_NAME即可。
- CHANNEL_ID: 取一個自己想要的名字就好，這個會出現在最後產出的新網址上。
- —expires 1d: 1d就是一天，看要設個12h, 7d, 2w都可，就算現在設完想要改也可以直接去firebase hosting後台改。

## 部署完成

部署完後就會拿到新網址，[https://aler-dev--[bbb]-[ccc].web.app](https://kymco-tw-dealer-dev--preview-nl7nz91f.web.app/)，

- aler-dev: 就是原本部署指定的TARGET_NAME所指向的hosting
- bbb: 就是CHANNEL_ID，也就是preview
- ccc: 自動產的亂數隨機碼

hosting後台也可以看到多了一個channel如下圖。
![preview_channel_success](/firebase-hosting-preview-channel/preview_channel_success.png)






