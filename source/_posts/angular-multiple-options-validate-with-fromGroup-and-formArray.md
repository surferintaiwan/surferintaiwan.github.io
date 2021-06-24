---
title: angular-multiple-options-validate-with-fromGroup-and-formArray
date: 2021-05-13 23:45:19
tags:
- angular
- javascript
- angular stepper
- angular formarray
- angular formgroup
categories:
- angular
---
[Demo Code](https://stackblitz.com/edit/angular-multiple-options-validate)

最近在寫一個多步驟的問卷功能，而問卷其中一個步驟的其中一題是必填，但該題是多選又要限制使用者最多只能選三個選項，上網查了有點久原來可以用fomArray來處理。

## **一、使用mat-stepper建立步驟的驗證**

### **1.import要用到的幾個套件(記得在ngModule的imports要記得引入)**

```
// app.module.ts
import { FormsModule } from '@angular/forms';
import { ReactiveFormsModule } from '@angular/forms';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations'
```

### **2.建立template**(裡面有一些變數到後面會提到)

這個部分不特別說明了，就是建立三個步驟的問卷，而我把主要要demo用的複選題放在了步驟1。

```
// app.component.html
<mat-horizontal-stepper [linear]="isLinear" #stepper>
  <mat-step [stepControl]="firstFormGroup">
    <form [formGroup]="firstFormGroup">
      <ng-template matStepLabel>Fill out your name</ng-template>
      <mat-form-field>
        <mat-label>Name</mat-label>
        <input matInput placeholder="Last name, First name" formControlName="firstCtrl" required>
      </mat-form-field>
      <div *ngFor="let option of a1OptionsToArray; let i=index">
        <input type="checkbox" [value]="option" (change)="[a1CheckboxChange($event),a1OptionLimitTo3()]"
        [disabled]="a1Disabled && !a1OptionsToObject[option]['checked']">
        {{a1OptionsToObject[option]['name']}}
      </div>
      <div>
        <button mat-button matStepperNext>Next</button>
      </div>
    </form>
  </mat-step>
  <mat-step [stepControl]="secondFormGroup" label="Fill out your address">
    <form [formGroup]="secondFormGroup">
      <mat-form-field>
        <mat-label>Address</mat-label>
        <input matInput formControlName="secondCtrl" placeholder="Ex. 1 Main St, New York, NY"
               required>
      </mat-form-field>
      <div>
        <button mat-button matStepperPrevious>Back</button>
        <button mat-button matStepperNext>Next</button>
      </div>
    </form>
  </mat-step>
  <mat-step>
    <ng-template matStepLabel>Done</ng-template>
    <p>You are now done.</p>
    <div>
      <button mat-button matStepperPrevious>Back</button>
      <button mat-button>Reset</button>
    </div>
  </mat-step>
</mat-horizontal-stepper>
```

### **3. 建立formGroup並設定好要驗證的欄位**

雖然有三個步驟，但只有前面兩個步驟需要欄位驗證，所以建立兩個formGroup，其中有發現還多了個a1CheckedArray比較特別，等於是在formGroup下建了一個a1CheckedArray欄位且必須驗證是否任何一個選項有被勾選了才能驗證通過。

```
// app.component.ts
ngOnInit() {
    this.firstFormGroup = this._formBuilder.group({
      firstCtrl: ['', Validators.required],
      a1CheckdArray: this._formBuilder.array([], Validators.required),
    })
    this.secondFormGroup = this._formBuilder.group({
      secondCtrl: ['', Validators.required]
    });
  }
```

## **二、透過typescript的interface導入問卷題目，並且轉成後續要用的陣列跟物件**

因為問卷的選項未來可能會常常改，所以選項的部分不是寫死在HTML裡面，而是寫在interface上方便未來維護，這一段可能要花點時間理解，會有點繞喔~

### **1.建立interface並寫兩個get方法，分別取用interface這個物件的keys跟values給後面要用**

```
// app.component.ts
enum OrderA1 {
  opt1 = '舒適性',
  opt2 = '續航力',
  opt3 = '價格經濟',
  opt4 = '外型亮眼',
  opt5 = '其他',
}

get OrderA1Keys() { return Object.keys(OrderA1) }
get OrderA1() { return Object.values(OrderA1); }
```

### **2.把從interface拿到的陣列存到this.a1OptionsArray這個變數中**

this.a1OptionsArray最後會是長這樣: [‘op1’,op2,…]這會用在HTML那邊要透過ngFor列印時，塞value到每個選項的input使用，並且也用於要比對該選項是否有被checked是否需要disabled時使用。

```
// app.component.ts
a1OptionsToArray: Array<string>;
a1OptionsToObject: {
    [optionKey: string]: { name: string, checked: boolean }
  } = {};

ngOnInit() {
    this.a1OptionsToArray = this.OrderA1Keys;
    this.a1OptionsToArray.forEach(
      optionKey => {
        this.a1OptionsToObject[optionKey] = { name: OrderA1[optionKey], checked: false }
      }
    )
  }
```

### **3.再把這個陣列取出來加工，一個一個搭配checked屬性存到this.a1OptionsObject這個物件中。**

this.a1OptionsObject會是以原本的keys繼續當作key值，去組成新的物件，最後會是長這樣:{op1: {name:’舒適性’, checked: false}, op2: {name:’舒適性’, checked: false}…}這會用在兩個地方，HTML那邊來比對該選項是否有被checked是否需要disabled時使用；在ts邊則是用在比對出中文字存到formArray裡面。

```
// app.component.ts
a1OptionsToArray: Array<string>;
a1OptionsToObject: {
    [optionKey: string]: { name: string, checked: boolean }
  } = {};

ngOnInit() {
    this.a1OptionsToArray = this.OrderA1Keys;
    this.a1OptionsToArray.forEach(
      optionKey => {
        this.a1OptionsToObject[optionKey] = { name: OrderA1[optionKey], checked: false }
      }
    )
  }
```

## **三、建立a1CheckboxChange函式把選擇的選項結果直接轉成乾淨的字串陣列**

### **1.透過此一函式去獲得HTML那邊傳進來的value**

這個vlaue其實就會是a1OptionsToArray的op1或op2或op3…

### **2.判斷傳進來的是不是有被勾選checked了**

特別補充說明一下，這傳進來的checked跟我們前面在建立物件時對應的checked不一樣喔，他這個checked就是HTML自己原生的屬性checked，不要搞混了。

### **3.如果被勾選了，才會去把value丟進去前面建立的a1OptionsToObject物件比對出中文字，並塞到a1CheckdArray，到時候輸出整個formgroup的時候就會看到這一欄是可以方便取用的中文字陣列了。**

同時也會把該選項在物件內的checked寫成true，這樣HTML那邊的diabled才知道甚麼時候要把其他選項鎖起來。

### **4.如果沒有被勾選，就會拿出a1CheckdArray去跟現在送進來取消勾選的選項本身的中文字逐一做比對，如果有比對到第幾次，就相對能推算出相同的中文字是在陣列中的第幾位，就能順勢將a1CheckdArray裡的中文字刪除掉了。**

```
// app.component.ts
a1CheckboxChange(event) {
    const a1CheckdArray: FormArray = this.firstFormGroup.get('a1CheckdArray') as FormArray;
    const value = event.target.value;
    if (event.target.checked) {
      a1CheckdArray.push(new FormControl(this.a1OptionsToObject[value]['name']));
      this.a1OptionsToObject[value]['checked'] = true;
    } else {
      let i = 0;
      a1CheckdArray.controls.forEach((item: FormControl) => {
        if (item.value === this.a1OptionsToObject[value]['name']) {
          a1CheckdArray.removeAt(i);
          this.a1OptionsToObject[value]['checked'] = false;
          return;
        }
        i++;
      });
    }
  }
```

## **四、建立a1OptionLimitTo3限制最多只能選三個選項**

這邊概念就簡單多了，就是直接驗證前面建立的那個formGroup裡面那個a1CheckArray陣列裡面有被塞的值是不是已經滿三個了，如果是的話就會把整個欄位鎖起來，才能使其他本來在物件裡的checked是false的選項都被鎖起來無法選。

```
// app.component.ts
  a1OptionLimitTo3() {
    if (this.firstFormGroup.get('a1CheckdArray').value.length === 3) {
      this.a1Disabled = true;
    } else {
      this.a1Disabled = false;
    }
  }
```

以上說明如果看不懂，可能直接去拿我stackblitz上的那個code去操作會比較容易理解XD