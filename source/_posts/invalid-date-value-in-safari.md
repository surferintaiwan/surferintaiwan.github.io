---
title: invalid_date_value_in_safari
date: 2021-03-11 22:59:58
tags: 
- javascript
- safari
categories:
- javascript
---

因為先前有個新專案快要上線，PM就開始猛測試，發現某個表單用了iPhone的Safari去送出就會失敗，但Android手機的其他人用Chrome卻可以送出表單。

## 其實只要是Safari，手機遇到的問題，Mac上也會有

不過我們沒有iPhone的轉接線可以轉接到電腦上看到底是報了什麼錯，只好不斷改code不斷寫alert不斷部署上去，用iPhone看看alert出來的資料，最後發現有個日期的部分印出來是NaN，而該日期是透過new Date()產生的，因此就開始往new Date()跟Safari的方向研究。

(題外話，其實不用那麼蠢的說…因為我一開始聽信PM說Mac上的Safari沒問題，唯獨iPhone的Safari有問題，我只好一直不斷部署上去再用iPhone去測，最後當我找到問題時我才想到用一下Mac上的Safari去測一下，發現其實也是有問題的，如果我早點用Mac的Safari測，就可以馬上看到報錯是什麼，免去我抓問題抓那麼久Q”Q)

## 慎用new Date()

底下開始正式說明問題。我的需求是希望塞一個特定的日期時間進去new Date()，去產出一個我想要的date像是這樣`new Date('2013-05-13 03:00')`，但這樣的寫法在Chrome或其他瀏覽器上產出來的值確實是正確的date格式。

可到了Safari上就會轉不過去了，[上網查到這篇](https://stackoverflow.com/questions/16616950/date-function-returning-invalid-date-in-safari-and-firefox)，必須要寫成`new Date('2013-05-13T03:00:00Z')`，Safari才會順利讓new Date()產出我要的date格式。

## Safari對於new Date()使用較嚴格的感覺

總之是Safari比較嚴格，會需要你輸入new Date()裡面輸入的值要非常完整，至於Chrome之類其他的瀏覽器就沒那麼嚴格，輸入大致的時間格式就可以清楚幫你轉換了。

## 結論：日後有日產出特定date格式的需求，盡量以moment套件來做

但其實要補中間的T跟後面的Z我覺得有點麻煩，最後是直接以moment套件寫`moment('2013-05-13 03:00').toDate()`，去產出我要的date格式，而以moment套件產出的date經實測確實可以讓Safari或Chrome等等瀏覽器都可以正常運行。