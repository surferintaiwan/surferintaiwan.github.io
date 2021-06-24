---
title: 檢查iOS版本號是否為14
date: 2021-04-15 22:32:33
tags:
- javascript
- iOS
categories:
- javascript
---

最近在串Webex直播，但發現iOS 13以下(包含13)就都沒辦法顯示視訊畫面，所以必須判斷13以下的都要另外跳另外的alert，問了Google大神，看了幾篇終於找到以下這個寫法去判斷iOS版本。

基本上就是透過navigator.appVersion拿到使用者的裝置資料，而iOS的版本號都是3個數字組成，像是iPhone OS 4_3_3，所以必須用正則表示式把這三個數字切出來，至於我們要判斷的其實只要判斷第一個數字即可。

而navigator.platform那一段我先註解掉了，因為我不需要先判斷他是不是iPhone或iPod或iPad，我只要確定他的appVersion是OS開頭的即可。

參考連結: https://stackblitz.com/edit/angular-detect-ios-version-is-14



    function iOSversion() {
      // if (/iP(hone|od|ad)/.test(navigator.platform)) {
      // iOS version will be like iPhone OS 4_3_3, you will get three digits
      var v = navigator.appVersion.match(/OS (\d+)_(\d+)_?(\d+)?/);
      return [
        parseInt(v[1], 10),
        parseInt(v[2], 10),
        parseInt(v[3] || (0 as any), 10)
      ];
      // }
    }

    const ver = iOSversion();

    if (ver[0] >= 14) {
      alert("iOS version is" + ver);
    } else {
      alert("iOS version less than 14");
    }
