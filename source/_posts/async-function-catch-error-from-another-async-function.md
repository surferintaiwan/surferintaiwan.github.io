---
title: ASYNC FUNCTION再去AWAIT一個ASYNC FUNCTION，是否能夠順利CATCH到ERROR?
date: 2021-06-26 18:45:38
tags:
- javascript
categories:
- javascript
---
最近在寫功能的時候，常常會把一長串的程式封裝成幾個function，有的如果剛好裡面有需要await非同步的東西就需要再把該function包成async function，但就讓我產生一個疑問，究竟這樣多層的async await，在外層的catch能不能接到內層吐的error呢?

[Demo Code](https://stackblitz.com/edit/catcherror-for-another-asyncfunction?embed=1&file=index.js)

## 先把結論寫在前面

1. **答案是可以的。**
2. **發現另一件事，如果內層的async funciton本身已經有catch error掉了，外層的catch是不會接到error的，且外層後面的程式還是可以繼續往下走!!**

## 實驗
```
function promise1() {
  return new Promise((res, rej)=>{
    res('resolvePromise1')
  })
}

function promise2() {
    return new Promise((res, rej) => {
    setTimeout(()=>{
      rej('rejectPromise2')
    },5000)
  })
}

function promise3(){ 
  return new Promise((res, rej) => {
    setTimeout(()=>{
      rej('rejectPromise3')
    },5000)
  })
}

async function awaitPromise1AndTestPromise1AndPromise2() {
  try {
    const resultPromise1 = await promise1()
    console.log(resultPromise1)
    const resultPromise2 =  await awaitPromise2()
    const resultPromise3 = await awaitPromise3()
  } catch(error){
    console.log('外層catch=>', error)
  }
}

async function awaitPromise2() {
  try {
    return await promise2()
  } catch(error) {
    console.log('內層catch=>', error)
  }
}

async function awaitPromise3() {
    return await promise3()
}
```

這邊大概說明一下整個code:(但我覺得不用看說明，可能直接看code比較好懂)

1. 建立了三個promise，promise1只回resolve，promise2跟promise3只回reject，而重點會是在後兩者，因為我就是要看多層async function的try catch是怎麼接到error的。

2. 建了三個async function，awaitPromise1AndTestPromise1AndPromise2 > 拿來await另兩支async functionawaitPromise2 > 用來await promise2，並且自己有個catch在接errorawaitPromise2 > 用來await promise3，但自己沒有個catch在接error

3. 接著整份code跑出來結果會是下面截圖這樣，3-a. 當呼叫awaitPromise1AndTestPromise1AndPromise2後，3-b. 程式執行到await awaitPromise2()時，因為他本身就有自己的try catch，所以錯誤訊息先在內層被印出來了，**也就會印出內層catch=> rejectPromise2**這段字樣，**從這部分就可以知道原來內層catch如果把error接到了，外層的catch就不會接到error，並能把後面的程式繼續執行。**(但其實我在工作寫功能的時候，我幾乎都是會統一在最外層catch接error就是了，因為我不會希望某個await async funtion沒執行成功，但卻又執行另一個await async function)3-c. 程式不會因為前面的內層catch接到了error導致外層沒有執行，而是會繼續執行下去，所以執行到await awaitPromise3()時，外層的catch就會接到另一個error，**也就會印出外層catch=> rejectPromise3**這段字樣，**從這部分就可以知道原來外層catch是可以接到另一個async function吐出來的error的。**

https://stackblitz.com/edit/catcherror-for-another-asyncfunction?embed=1&file=index.js