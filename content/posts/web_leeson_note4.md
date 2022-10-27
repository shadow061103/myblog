---
title: "版型課程第四周筆記"
date: 2022-10-27T17:12:10+08:00
draft: true
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---

# 版型課程第四周筆記
### Layout
> 類似 .Net MVC的share.cshtml
### 用gulp整合
- 前端任務化
- ejs樣板語言
<%- content %>
- app資料夾:放css,html,images,js檔案
- dist資料夾:輸出在用沒事不要碰他，儲存就會放一份，把layout跟頁面合併在一起
    會自動監聽
- 改layout.ejs檔案就好
只是幫助管理
- [Layout說明](https://cacoo.com/diagrams/fWdDuMY0WrfI0im7/CD531?reload_rt=1598429242751_0)
- ejs include(路徑) 把外部檔案載入
- 檔案要設成相對路徑 githubpage才會讀得到
#### 連結到錨點
- 使用id選擇器
- a href連結要加#

### Sass預處理器
> 是一種css的擴充，解決重複性、可維護性差的問題
- 語法分為新的SCSS(Sassy CSS、Sass 3，檔名為 *.scss) 和舊的 SASS（不使用大括弧格式、使用縮排，不能直接使用 CSS 語法、學習曲線較高等特性，檔名為 *.sass）
- [PPT](https://docs.google.com/presentation/d/11-HFPxkmVj5b6WP50zkKB_GtccvynUu3GaDeALaLpd0/edit#slide=id.p42)
- 安裝套件 Live Sass Compiler
- [環境教學](https://ithelp.ithome.com.tw/m/articles/10202661)
- 可以幫助縮短語法[語法介紹](https://tw.alphacamp.co/blog/css-preprocessor-sass-scss)
- [github demo](https://github.com/shadow061103/hexschool_layout_hw/blob/master/week4/css/all.scss)
- 變數設計 一定要放在最上面
    - 前綴詞$ 
    - 可以做數值運算或色彩運算
    - lighten($primary,30%) 調亮
    - darken($primary,30%) 調暗
    - 通常用來存色彩或空間跟尺寸
    - 命名用情境色primary、secondary、accent 或功能background
    - 命名可以參考bootstrap 避免使用色名 green或yellow，尺寸盡可能語意化large-size
    
 ```
 主色 //變數
$primary:#ff0000;
 ```
- import檔案拆分
    - _variable.scss > all.scss >all.css
    - ``@import "variable"``
    - 檔名有_代表不會不輸出成css 拿來被合併用的(被忽略)
- .scss檔案會輸出成.css
    - 編譯完後vs code按watch sass 會輸出.css跟.css.map檔案，前者是轉譯過後的 CSS 檔案，後者是方便使用瀏覽器除錯工具除錯時連結原檔案和轉譯檔案，方便除錯
    - css階層越多，網頁渲染越慢，盡量不要超過4層
    - 斷點是寫在scss


### SCSS結構參考

```
@import "variable";// 變數  
@import "reset";  // reset.css  
@import "layout"; // 共同框架,第一層
@import "index";     
@import "page1";     

```
## Live Sass Complier
VS code設定輸出目錄
檔案>喜好設定>設定>工作區tab>右上角開啟設定

```
{
    "liveSassCompile.settings.formats": [
        {
            "format": "compressed",
            "extensionName": ".min.css",
            "savePath": "/css/"
        }
    ]
}
```

### githubpages路徑
github.io/project/dist
