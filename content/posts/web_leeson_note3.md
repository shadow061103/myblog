---
title: "版型課程第三周筆記"
date: 2022-10-27T17:12:08+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---

# 版型課程第三周筆記

## RWD自適應語法

### 依照顯示器寬度去呈現樣式

- 375px iphone7-11
- 414px
- ipad 768px~992px
- 767px 手機以下
- bootstrap 1140px
- media要從最大寫到最小
- 潛規則 >去客戶那邊紀錄裝置解析度/瀏覽器/手機型號
``` css
.list .item{
    width:25%;
    }
@media (max-width :992px)
{
.list .item{
    width:50%;
    }
}
@media (max-width :767px)
{
.list .item{
    width:100%;
    }
}
```
* 響應式要加
`<meta name="viewport" content="width=device-width, initial-scale=1.0">`
* CSS 語法 media queries
`@media(max-width:768px){}`

    - 可以設定限定方式 有需要再查

## css權重
- 先看權重再看先後順序，後者贏
- HTML標籤 1分
`lang=css tag { color:black; }`
- 類別選擇器10分
`.class { color:black; }`
- id選擇器100分
`#id { color:black; }`
- 行內樣式1000分
`<h1 style="color:black;"></h1>`
- !important 10000分
`tag { color: black !important;}`
- 權重如果一樣，看css檔案裡的先後順序
- 永遠不要用id (**當網頁內錨點用 a>href**)

### 網頁不可以顯示X軸(左右滑動)
x 軸：網頁的水平 scroll bar

**使用max-width**
- 自適應網頁寬度
- css reset加這段語法
```css
img{
    max-width:100%;
    height:auto;
}
```
- 全域border box

```css
*,*::before,*.::after{
box-sizing:border-box;
}
```
- 寬度使用%
 加起來不超過100%

- 主要內容與背景圖的差異
    - 背景圖也需要設計
    - 背景圖 “不可” 與主要內容混在一起
        - 扣分：
            - 背景圖缺少延伸
            - 出現水平捲軸
- 圖片：
    - 應分辨 img 與 background img 的運用差異
        1. 長寬比的切換
        2. 圖片是否需要裁切

### Flex 拆解觀念
- flex-grow、flex-shrink
    - [空間計算](https://wcc723.github.io/css/2020/03/08/flex-size/)

## 排版
float > flex > grid
- float:水平
- flex:水平+垂直
- grid:面
- 滿寬
    - 左右向外拉 用margin
- max-width 限制最大寬
- width:強制寬度


## 網頁寬度單位
- px：絕對單位，代表螢幕中每個「點」( pixel )。
- em：相對單位，每個子元素透過「倍數」乘以父元素的 px 值。
- rem：相對單位，每個元素透過「倍數」乘以根元素的 px 值。
- %：相對單位，每個子元素透過「百分比」乘以父元素的 px 值。
