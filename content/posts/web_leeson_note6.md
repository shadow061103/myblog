---
title: "版型課程第六周筆記"
date: 2022-10-27T17:12:15+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---

bootstrap是手機寫到PC

## 格線系統
> 避免寫一些奇怪數字 俗稱magic number
```css
.col-1{
width:60px;
}
.col-2{
width:120px;
}
```
- 格線系統關鍵字：column(欄)、gutter(間距)
    - 左右會用margin推 
- [960 grid](https://960.gs/demo.html) 要符合1024*768解析度 通常會有12欄(比較好整除)
- 沒格線系統就會用magic number
- bootstrap是用padding推gutter的值
- column內不能多用padding跟margin，因為寬度會超過%數
- row裡面再包column，因為元素才會對齊
- 沒用格線系統只有局部區塊好看，用的話一致性高有規律
- class=row的第一層只能放col

### bootstrap的斷點設計
- [文件位置](https://bootstrap.hexschool.com/docs/4.2/layout/grid/#grid-options)
- 依照裝置大小去置換class
> 常見錯誤，在 .col 增加寬度
> 常見錯誤，在格線系統調整左右 margin 與 padding
> 常見觀念：可以加上下 的 margin 與 padding
> 常見觀念：最外層至少補一個 container
> 常見觀念：整體格線邏輯是一致

### 匯入bootstrap的module
- [位置](https://bootstrap.hexschool.com/docs/4.2/getting-started/theming/#importing)
```css
// Required
@import "../node_modules/bootstrap/scss/functions";
@import "../node_modules/bootstrap/scss/variables";
@import "../node_modules/bootstrap/scss/mixins";

// Optional
@import "../node_modules/bootstrap/scss/reboot";
@import "../node_modules/bootstrap/scss/type";
@import "../node_modules/bootstrap/scss/images";
@import "../node_modules/bootstrap/scss/code";
@import "../node_modules/bootstrap/scss/grid";
```
- 可以在裡面直接改樣式
    - spacers:推間距
    - $theme-colors:主要顏色
- !default 誰先註冊就是預設值，看先後會不會被覆蓋，但如果是一般樣式後面沒寫，就會覆蓋掉default
- utilities是一定要用的(工具類別)
 
### 編譯 [BS4 SCSS](https://github.com/twbs/bootstrap/tree/master/scss) ，用 [NPM 套件工具](https://bootstrap.hexschool.com/docs/4.2/getting-started/download/#npm)來安裝，[匯入方式](https://bootstrap.hexschool.com/docs/4.2/getting-started/theming/#importing)
步驟一：開啟新資料夾，將資料夾 cd 拉進終端機
步驟二：輸入指令 `npm init`，一直按 enter 後，資料夾裡面會多一個 `package.json` 檔
步驟三：安裝 BS4 套件，指令是 `npm install bootstrap`，安裝成功後會多一個 `node_modules` 資料夾
步驟四：@import BS4 SCSS 檔案，[連結處](https://bootstrap.hexschool.com/docs/4.2/getting-started/theming/#importing)

> 不要去修改 node_modules 裡面的檔案
> 需要使用到的項目直接 @import node_modules 的檔案
> 需要變更內容，請抓出來一份來進行 SCSS 編輯，並設為預設值 !default

### 結構
> 注意事項: 自己另外寫的樣式的載入順序 import 在 BS4 的 SCSS 後面
#### BS4 載入方式 1 - 全部載入(只想修改變數樣式)

```
// Required
@import "../node_modules/bootstrap/scss/functions";
@import "./helpers/variables"; // 複製一份 variable 來覆蓋
@import "../node_modules/bootstrap/scss/mixins";
@import "../node_modules/bootstrap/scss/bootstrap";
```

#### BS4 載入方式 2 - 載入網格系統、工具類(通用類別)、卡片、分頁、導覽列、Modal、表單、表格與按鈕

```
// Required
@import "../node_modules/bootstrap/scss/functions";
@import "./helpers/variables"; // 複製一份 variable 來覆蓋
@import "../node_modules/bootstrap/scss/mixins";
// Option
@import "../node_modules/bootstrap/scss/grid";
@import "../node_modules/bootstrap/scss/utilities";
@import "../node_modules/bootstrap/scss/card";
@import "../node_modules/bootstrap/scss/pagination";
@import "../node_modules/bootstrap/scss/navbar";
@import "../node_modules/bootstrap/scss/modal";
@import "../node_modules/bootstrap/scss/forms";
@import "../node_modules/bootstrap/scss/tables";
@import "../node_modules/bootstrap/scss/buttons";
```

#### BS4 載入方式 3 - 只想載入 button

```
// Required
@import "../node_modules/bootstrap/scss/functions";
@import "./helpers/variables"; // 複製一份 variable 來覆蓋
@import "../node_modules/bootstrap/scss/mixins";
// Option
@import "../node_modules/bootstrap/scss/buttons";
```

#### BS4 載入方式 4 - 載入網格系統與工具類(通用類別)
```
// Required
@import "../node_modules/bootstrap/scss/functions";
@import "./helpers/variables"; // 複製一份 variable 來覆蓋
@import "../node_modules/bootstrap/scss/mixins";
// Option
@import "../node_modules/bootstrap/scss/grid";
@import "../node_modules/bootstrap/scss/utilities";
```

#### BS4 載入方式 5  - 單純只想載入工具類(通用類別)

```
// Required
@import "../node_modules/bootstrap/scss/functions";
@import "./helpers/variables"; // 複製一份 variable 來覆蓋
@import "../node_modules/bootstrap/scss/mixins";
// Option
@import "../node_modules/bootstrap/scss/utilities";
```