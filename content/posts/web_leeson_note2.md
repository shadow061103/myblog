---
title: "版型課程第二周筆記"
date: 2022-10-27T17:12:06+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---


## Flex
- container
    - 1.內元件要有效果，就要在外容器加上 display:flex
    - 2.每個 HTML 標籤，能同時擁有內元件跟外容器身份
- Flex 主軸與交錯軸觀念([測試工具](https://codepen.io/peiqun/pen/WYzzYX))

## 外容器語法
- 決定軸線：flex-direction (互動詢問)
    - 1.row (**預設**)
    - 2.row-reverse
    - 3.column
    - 4.column-reverse
- 主軸對齊：justify-content 
    - 1.flex-start (**預設**)
    - 2.center
    - 3.flex-end
    - 4.space-between
    - 5.space-around
    - 6.space-evenly
- 換行屬性：flex-wrap
    - 1.nowrap
    - 2.wrap (寬度不夠會自動往下長)

- 交錯軸對齊：align-item
    - 1.flex-start
    - 2.center
    - 3.flex-end
    - 4.stretch
    - 5.baseline

### 圖片設定
- 背景圖片
    - background-image:url(../img/logo.png)
- 圖片重複顯示
    - background-repeat: no-repeat不重複 /repeat-x水平重複 / repeat-y垂直重複 
- 背景顏色 background-color: #色碼
- 背景圖案會蓋在背景顏色上面
- 背景圖片位置 background-position:right(x軸) bottom(y軸) /30px 30px
- backbround-size:cover;

### font-weight 屬性用法介紹
CSS 中設定文字粗細（也可以說是字體厚度）的語法是 font-weight，可以是 100~900 的數字或 normal、bold、bolder、lighter ... 等値。
- normal：也就是預設字體厚度，其實可以不用特別寫出來。
- bold：常用的粗體字。
- bolder：比粗體更粗一點。
- lighter：比一般字體更細。
- 100~900：數字越大越厚，數字小於 500 似乎效果不明顯。

### font-style 屬性
CSS font-style 屬性的功能是用來設計網頁文字的斜體、傾斜等字體樣式，實際效果與 HTML 的 <i></i> 標籤類似，簡單來說就是網頁文字的斜體字特效在 CSS 的表示法，CSS font-style 屬性提供兩種類似的斜體字設定方式，均獲得大部份主流瀏覽器的支援。

```
font-style: 字體樣式設定值;
```
- normal:預設值，讓瀏覽器根據標準文字樣式顯示字體
- italic :將文字樣式設為斜體字。
- oblique:將文字樣式設為傾斜字體，與 italic 幾乎一樣。
- initial:將文字樣式設為預設值。
- inherit:繼承自父層的 font-style 樣式設定，非所有瀏覽器支援，不建議使用。

## Flex補充
 - [圖解：CSS Flex 屬性一點也不難](https://wcc723.github.io/css/2017/07/21/css-flex/)

