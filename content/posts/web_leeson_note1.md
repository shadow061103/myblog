---
title: "版型課程第一周筆記"
date: 2022-10-27T17:12:03+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---


## css reset
可以reset讓瀏覽器一致
[css reset ](https://meyerweb.com/eric/tools/css/reset/)

## 加速寫CODE的速度
* 練習英打，若沒時間最少練習常用的單字，可參考   透過 keybr.com [*練習內容*](https://github.com/hexschool/EmmetPractice)
* 使用套件 TabNine  記憶單辭。

## 容器
### 區塊元素
- 預設display:block
- 盡可能占滿整個版面
- 會另起一行進行呈現
- 可以設定寬高
- div、p、h1~h6、
ul、ol、li、
dl、dt、dd、
form、table、hr、
blockquote 、
address、menu

### 行內元素
- display inline
- 比較常用在段落裡
- 不能設定寬高 除非改成區塊元素**display:block**
- span、em、i、b、strong、a、img、input、br、select、textarea、q、bdo、sub、sup

### 邊界線
``` css=
{
border: 1px solid blue;
}
```

說明:
- 線條粗細
- 樣式 solid實心線 dashed虛線 dotted點點
- 顏色

### 留白
- padding
往內推元素
- margin
往外推元素
盡量要一致

### 水平置中
是整個元素置中
```css =
{
margin-left: auto;
margin-right: auto;
margin:0 auto;
}
```
文字內容置中
```css=
{
text-align:center;
}
```



## 盒模型
元素寬高會包含 border
```css=
{
box-sizing: border-box;
}
```


### Flex設定對齊
- 主軸對齊
**justify-content**
flex-start / flex-end / space-between貼齊容器

- flex-wrap: wrap斷行
讓他不會自適應縮小寬度

- 交錯軸線的對齊
**align-items**
跟justify-content一樣可以設定

### VS code套件 
**preview on web server**
預覽網頁
可以即時更新

### 設計工具
- skitch
  設計版面 可以幫助思考
- markman
  設計寬高&顏色


### 偽元素選取器
.table tr:hover ~tr {}
之後所有tr

- ::before 緊貼這個元素的最前面
- ::after  緊貼這個元素的最後面

### 定位
position預設是static
不會被特地排版，而是照瀏覽器的配置自動排版在頁面


- 相對定位
  ```css
  {
  position:relative
  }
  ```
  裡面的元素位置都會相對於它
  可以設定top、bottom、right、left
  元素會重疊
  
- 絕對定位
  ```css
  {
  position:absolute
  }
  ```
會根據父元素定位點設定
可以設定負值
可以讓元素重疊在一起
z-index設定權重 數字越大會在越上面
其他元素會補上它原本的位置

當元素的position設定absolute後，它就會往外層的元素找是否有position:relative | absolute | fixed | inherit(若繼承的是前面3個之一)的元素，若是都沒有，就會以該網頁頁面(<body>)的左上角為定位點。

[程式說明](https://ithelp.ithome.com.tw/articles/10212202)


### line-height
```css=
p {
    font-size: 16px;
    line-height: 24px;
    margin-bottom:30px;
}
```
> 行高24px,字體16px置中
> 與底下的差距為(24-16)/2+30 = 34px
> 

### img 下面留白一點點 2~3px ?!
清除方法:
* img 加 display:block or vertical-align:middle

### emmet練習
[emmet](https://docs.emmet.io/cheat-sheet/)