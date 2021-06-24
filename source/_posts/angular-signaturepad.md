---
title: angular簽名板
date: 2021-04-16 21:51:27
tags:
- angular
- javascript
- signaturepad
categories:
- angular
---
因為需要寫一個頁面，裡面會有個按鍵按下後，會跳出一個彈窗就包著簽名板，該簽名板上會提供確認送出以及清除的功能。

基本有建立了兩個元件，一個是demopage，一個是signaturepad。
就是在demopage元件寫一個打開簽名板的按鍵，該按鍵按下去後，就會透過MatDialog以pop-up方式打開sinaturepad元件。

demo連結:https://stackblitz.com/edit/angular-signaturepad?file=src%2Fapp%2Fdemopage%2Fdemopage.component.html

(打開後會發現套件會報錯，但如果把code弄到自己的angular專案上是可以成功運行的，所以這個demo連結就當作放code的地方就好，沒有實際demo的效果。)

## 在demopage上

### 寫一個按鍵透過MatDialog以pop-up方式打開signaturepad元件

如前所述，我的需求是在既有頁面上以pop-up方式開一個簽名板。

### 將圖檔從base64轉成blob

之所以要轉成blob是因為我要上傳到firebase storage上時不能是base64，所以當signaturepad元件關閉時，他送過來我這裡的會是base64檔，因此必須使用ngx-image-cropper裡面的base64ToFile來轉成blob檔。

### 程式碼

```jsx
// html
<button (click)="openSignatureDialog()">Open SignaturePad</button>

// javascript
import { Component, OnInit } from "@angular/core";
import { MatDialog, MatDialogConfig } from "@angular/material/dialog";
import { base64ToFile } from "ngx-image-cropper";
import { SignaturepadComponent } from "../signaturepad/signaturepad.component";

@Component({
  selector: "app-demopage",
  templateUrl: "./demopage.component.html",
  styleUrls: ["./demopage.component.css"]
})
export class DemopageComponent implements OnInit {
  constructor(public dialog: MatDialog) {}

  ngOnInit() {}

  openSignatureDialog() {
    const dialogConfig = new MatDialogConfig();
    dialogConfig.disableClose = false;

    const dialogRef = this.dialog.open(SignaturepadComponent, dialogConfig);
    // 接收來自signaturepad元件emit出來的資料
    dialogRef.afterClosed().subscribe(async (imageDataBase64: string) => {
      // 轉成blob檔並用新增的service上傳成圖檔
      const blob = base64ToFile(imageDataBase64);
      console.log(blob);
    });
  }
}
```

## 在signaturepad元件上

### 引入angular2-signaturepad的SignaturePad

### 設定好簽名板的大小

簽名板本身的基底其實就是html的canvas畫布，至於畫布的大小就看要設多大多小？
是在signaturePadOptions輸入參數設定。

### 在ngAfterViewInit

這裡就幾乎都是[按照官方說的了](https://www.npmjs.com/package/angular2-signaturepad)。

不過比較重要的是在drawComplete函式時，我有特別把畫布上的圖存成另一個變數this.imageDataBase64，方便我要傳給demopage元件時用。

### 程式碼

```jsx
// html
<div class="closeBtn" (click)="closeDialog()">X</div>

<h2>簽名板</h2>
<div class="board">
  <signature-pad #signatureCanvas [options]="signaturePadOptions" id="signatureCanvas" (onBeginEvent)="drawStart()"
    (onEndEvent)="drawComplete()">
  </signature-pad>
</div>

<div class="stepperA text-align-center  buttonBlock">
  <button mat-button (click)="emitToFather()" class="prevStep">確認</button>
  <button (click)="clear()" class="clear">清除</button>
</div>

// javascript
import { Component, ViewChild, AfterViewInit } from "@angular/core";
import { SignaturePad } from "angular2-signaturepad/signature-pad";
import { MatDialogRef } from "@angular/material/dialog";

@Component({
  selector: "app-signaturepad",
  templateUrl: "./signaturepad.component.html",
  styleUrls: ["./signaturepad.component.css"]
})
export class SignaturepadComponent implements AfterViewInit {
  public imageDataBase64: string;
  public signaturePadOptions: object = {
    // passed through to szimek/signature_pad constructor
    minWidth: 1,
    canvasWidth: 800,
    canvasHeight: 400,
    backgroundColor: "rgb(255, 255, 255)",
    penColor: "rgb(0, 0, 0)"
  };
  @ViewChild("signatureCanvas", { static: true }) signaturePad: SignaturePad;

  constructor(private dialogRef: MatDialogRef<SignaturepadComponent>) {}

  // 以下全都是針對手寫板寫的
  ngAfterViewInit() {
    // this.signaturePad is now available
    // this.signaturePad.set('minWidth', 1); // set szimek/signature_pad options at runtime
    if (window.innerWidth <= 768) {
      this.signaturePad.set("canvasWidth", window.innerWidth - 40);
      this.signaturePad.set("canvasHeight", window.innerHeight - 250);
    } else {
      this.signaturePad.set("canvasWidth", 800);
      this.signaturePad.set("canvasHeight", 400);
    }
    this.signaturePad.clear(); // invoke functions from szimek/signature_pad API
  }

  drawComplete() {
    this.imageDataBase64 = this.signaturePad.toDataURL("image/jpeg");
    // will be notified of szimek/signature_pad's onEnd event
  }

  drawStart() {
    // will be notified of szimek/signature_pad's onBegin event
    console.log("begin drawing");
  }

  clear() {
    this.signaturePad.clear();
    this.imageDataBase64 = "";
  }

  emitToFather() {
    this.dialogRef.close(this.imageDataBase64);
  }

  closeDialog() {
    this.dialogRef.close();
  }
}
```

### * 一直遇到簽名檔無法清除的問題

這個查了有點久，一直刪不掉簽名檔，[最後發現其他人有提到要改寫法](https://forum.ionicframework.com/t/cannot-read-property-todataurl-of-undefined-signaturepad/186212)。
而我提供的demo code則是已經按照他們說的改好了，也確實可行喔。

## 在app.module上

要記得引入angular2-signaturePad。

```jsx
.
.
.
import { SignaturePadModule } from "angular2-signaturepad";

@NgModule({
  imports: [
		.
		.
		.
    SignaturePadModule
  ],
  .
	.
	.
})
```