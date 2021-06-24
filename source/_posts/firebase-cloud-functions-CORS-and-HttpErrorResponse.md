---
title: firebase Cloud Functions onRequest function出現CORS及HttpErrorResponse
date: 2021-05-13 23:18:06
tags:
- firebase
- firebase functions
categories:
- firebase
---
我有在Cloud Functions上寫了一支OnRequest的API，要讓我的網頁去呼叫，並且會回傳Excel檔案讓我下載，可是一開始就出現了CORS問題，最後又出現HttpErrorResponse問題，整整花了一個下午才解決。

但這是我第一次寫OnRequest的API，以前functions上都是架OnCall API，或在Cloud Run上面架API。

## 解決CORS⇒以Axios解決

![CORS](/firebase-cloud-functions-CORS-and-HttpErrorResponse/CORS.png)

一開始以為是cloud functions那邊CORS沒寫好，但其實我有裝CORS套件，在我的API裡面也有寫res.set()。

```jsx
// index.ts
import * as cors from 'cors';
const app = express();
app.use(cors({ origin: true }));

// 在我的API上也有寫res.set()
res.set('Access-Control-Allow-Origin', '*');
res.set('Access-Control-Allow-Methods', 'POST');
```

而我一開始也都是用HttpClient去呼叫。

```jsx
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http';

await this.httpClient.post(
      `https://xxx.com/download_something`,
      {
        name: 123
      }).toPromise();
```

最後轉換思考方向，改用Axios來呼叫API，就沒有CORS問題了，如下。

```jsx
await this.ajaxPost('https://xxx.com/download_something',{name:123})
```

## 有解決CORS問題，但還是沒辦法下載Excel，報HttpErrorResponse⇒以form.submit()解決

雖然CORS解決了，可是還是報HttpErrorResponse，如下圖。

![httpErrorResponse](
/firebase-cloud-functions-CORS-and-HttpErrorResponse/httpErrorResponse.png)

這時候想到以前用form.submit()呼叫onRequest function去下載Excel，都可以順利下載到檔案，那就乾脆改用form.submit()吧。

最後就成功了，可以順利下載Excel！！！

```jsx
function submitForm(url: string, data: any) {
    const formName = 'DynamicForms';

    const form = document.createElement('FORM') as HTMLFormElement;
    form.name = formName;
    form.action = url;
    form.method = 'POST';

    const formInput: HTMLInputElement[] = [];

    let count = 0;
    _.forEach(data, (item, name) => {
      formInput[count] = document.createElement('INPUT') as HTMLInputElement;
      formInput[count].type = 'HIDDEN';
      formInput[count].name = name;
      formInput[count].value = item.toString();
      form.appendChild(formInput[count]);
      ++count;
    });

    document.body.appendChild(form);
    form.submit();
    document.querySelector(`form[name=${formName}]`).remove();
  }

this.submitDynamicForm('https://xxx.com/download_something',{name: 123})
```

## 補充說明: 後端如何產出Excel檔案

怕忘記，紀錄一下。

```jsx
import xlsx from 'node-xlsx';	

res.set('Access-Control-Allow-Origin', '*');
res.set('Access-Control-Allow-Methods', 'POST');

const titleList = 每欄標題的陣列資料;
const titleWidthList = 寬度陣列資料;

const dataList = 這個是雙重陣列資料，就是陣列裡面的每個元素又都是陣列;
const fileName = 下載時的檔案名稱;
const options = { '!cols': titleWidthList.map(o => { return { wch: o } }) };
const result = xlsx.build([{ name: 'sheet', data: [titleList, ...dataList] }], options);

res.setHeader('Content-Type', 'application/vnd.openxmlformats');
res.setHeader("Content-Disposition", `attachment; filename=${fileName}.xlsx`);
res.end(result, 'binary');
```