---
title: dialogflow在heroku上的環境變數設定
date: 2021-06-26 17:59:05
tags:
- dialogflow
- heroku
categories:
- chatbot
---
## 使用dialogflow SDK一直報錯UNAUTHENTICATED

最近在寫LINE chatbot並搭配dialogflow來判斷使用者輸入文字意圖，在本地跑是正常的可以獲得detectIntent後的回應，但是在串dialogflow的時候就一直出現UNAUTHENTICATED，查了半天終於找到原因，是private_key讀取有問題。
![herokuo-config-variable-for-dialogflow-privatekey](/herokuo-config-variable-for-dialogflow-privatekey/UNAUTHENTICATED-in-dialogflow.png)

## 原來是heroku對於環境變數中\n的判斷會有問題

不過我去查本地環境變數設的private_key和Heroku環境變數設的private_key是一樣的，再輾轉查到了一篇才知道是因為PRIVATE_KEY本身都是長大概這樣`----BEGIN PRIVATE KEY-----\nMY-PRIVATE-KEY\n-----END PRIVATE KEY-----\n`，而裡面的`\n`字元無法被判斷是什麼，所以就要經過`(process.env.DIALOGFLOW_PRIVATE_KEY as string).replace(/\\n/g, '\n')`去處理，最後也終於可以正常串接dialogflow了。

```jsx
// dialogflow
const languageCode = 'zh-TW';
const projectId = 'my-line-chatbot-yutq';
const credentials = {
    client_email: process.env.DIALOGFLOW_CLIENT_EMAIL,
    private_key: (process.env.DIALOGFLOW_PRIVATE_KEY as string).replace(/\\n/g, '\n')
}

const sessionClient = new dialogflow.SessionsClient({ projectId, credentials });
```

參考資料：[https://stackoverflow.com/questions/39492587/escaping-issue-with-firebase-privatekey-as-a-heroku-config-variable/41044630](https://stackoverflow.com/questions/39492587/escaping-issue-with-firebase-privatekey-as-a-heroku-config-variable/41044630)