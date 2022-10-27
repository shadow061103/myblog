---
title: "版型課程第五周筆記"
date: 2022-10-27T17:12:12+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---

# 版型課程第五周筆記
### 表單三要素
- form
- input
- submit

### 使用id差別 為了使用者體驗
input使用id標註，用label for指定，只要點到label就會自動focus在input上
`<label for="Email">請輸入email:</label>`
### select選單
- 彈跳樣式跟顏色不要改，每個瀏覽器的樣式也不一樣，除非用div+js自幹
- 可以用套件
    - 可能下載過慢，太大包
    - 流量比例:7成圖片、2成css跟js、1成文檔 

#### disable的標籤資料不會送到後端，readonly的會

## css reset 差異
- [meyerweb CSS Reset](https://meyerweb.com/eric/tools/css/reset/)
    - 設計師不會用bootstrap 
     
- [normalize](https://necolas.github.io/normalize.css/)
    - bootstrap内建使用
    - ul、li樣式不一樣  
-  不能同時使用
- bootstrap的reset css https://github.com/twbs/bootstrap/blob/main/scss/_reboot.scss
### body跟*的差別
body是範圍內，*是每個tag標籤

### css先放再放js檔案，放在body最後面是最保險的
- 可以先做自己的util scss 換另外個專案
工具包的概念
- 純手寫
- 拉出BS4喜歡的功能再自己手寫
- 看懂BS4+都用BS4 scss去改寫

### 視窗置底技巧
> 讓元素根據視窗顯示的高度置底
- 用定位的方法也可以置底，但是把高度變小就會蓋過其它元素
- 可以在上層div用` min-height: 100vh;`讓元素隨視窗高度變化
- 100vh表示100%瀏覽器高度,20vh表示20%高度

### Material Icon
- [css連結](https://fonts.googleapis.com/icon?family=Material+Icons)
- 在要掛icon的地方插入
```
 <span class="material-icons">
        insert_chart
 </span>
```