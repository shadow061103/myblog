---
title: "版型課程第七周筆記"
date: 2022-10-27T17:12:18+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---

# 視差滾動
## Css3 animation
- [範例](https://codepen.io/liao/pen/JjYwNVW?editors=1100)
- 用keyframes去自訂動畫效果 from開始 to結束
```css
@keyframes example {
  from {left:0px; top:0px;}
  to {left:200px; top:0px;}
}
div {
/*  要執行的 animation 名稱   */
  animation-name: example;
/*  執行過程秒數  */
  animation-duration: 10s; 
}
```
- 查瀏覽器兼容效果[can I use](https://caniuse.com/?search=animation%20css)
- 也可以設定比例 %數代表時間比例
```css
@keyframes example {
  0%   {background-color:pink; left:0px; top:0px;}
  25%  {background-color:yellow; left:200px; top:0px;}
  50%  {background-color:blue; left:200px; top:200px;}
  75%  {background-color:green; left:0px; top:200px;}
  100% {background-color:red; left:0px; top:50px;}
}

/* The element to apply the animation to */
div {
  width: 100px;
  height: 100px;
  position: relative;
  background-color: red;
  animation-name: example;
  animation-duration: 4s;
/*  晚幾秒發動  */
  animation-delay: 2s; 
/*  要重複跑幾次，或者是用 infinite 一直持續跑   */
   animation-iteration-count: infinite; 
  
/*  停留在哪個影格  forwards、backwards、both */
/*   animation-fill-mode: backwards; */
/* 
  forwards 停留在最後一個位置   
  backwards: 留在第一個位置
  both 擁有前兩者功能
  */
}
```

### Animate.css
> [官網](https://animate.style/)
- 要載入CDN
- 基本用法
```<h1 class="animate__animated animate__bounce">An animated element</h1>```
- delay時間 css加animate__delay-2s
- 要複寫的話要有name跟keyframe
- [demo](https://codepen.io/shadow061103/pen/QWEWbmV)
- [基本範例](https://codepen.io/liao/pen/NWGevLa)
- [示範表](https://codepen.io/strapro/pen/dIqAH?editors=1010)
### transform動畫效果
- 效能比較好，CPU加速，比起position absolute(排版用)
- z軸單位越多，表示離越遠(畫面上更小)
- [範例](https://codepen.io/liao/pen/VwvqWZQ)
- 旋轉:transform: rotate(20deg);
- 放大倍率:transform: scale(2);
- 移動:transform: translateZ(0);
    - translateX,translateY,translate3d 
### 隱藏元素
- opacity:0;
- display:none; 只有這個不會保留元素空間
- visibility:hidden;
- transform 

### AOS
> [官網](https://michalsnik.github.io/aos/)
> [文件](https://github.com/michalsnik/aos)
- 可以跟animate.css整合在一起
- 效果要一致
- css加上data-aos="fade-right"
- 元素會被opacity設成透明
- init()
    - offset:設定出現時跟瀏覽器底端的距離 
- AOS做視差滾動 設定offset觸發效果，animate單純做動畫
- offset去偵測是否符合條件，增加class進去
- 可以跟animate.css一起使用
    - [整合](https://github.com/michalsnik/aos#integrating-external-css-animation-library-eg-animatecss)
    ```
      [data-aos] {
        visibility: hidden;
    }

    [data-aos].animate__animated {
        visibility: visible;
    }
    ```
#### 載入步驟
在 </body>前加上以下程式碼

```
<link rel="stylesheet" href="https://unpkg.com/aos@next/dist/aos.css" />
<script src="https://unpkg.com/aos@next/dist/aos.js"></script>
<script>
 AOS.init();
</script>
```


AOS 單一設計

```
<div
    data-aos="fade-up"
    data-aos-offset="200"
    data-aos-delay="50"
    data-aos-duration="1000"
    data-aos-easing="ease-in-out"
    data-aos-mirror="true"
    data-aos-once="false"
    data-aos-anchor-placement="top-center">
  </div>
```


AOS 全域設計

```
AOS.init({
  // Global settings:
  disable: false, // accepts following values: 'phone', 'tablet', 'mobile', boolean, expression or function
  startEvent: 'DOMContentLoaded', // name of the event dispatched on the document, that AOS should initialize on
  initClassName: 'aos-init', // class applied after initialization
  animatedClassName: 'aos-animate', // class applied on animation
  useClassNames: false, // if true, will add content of `data-aos` as classes on scroll
  disableMutationObserver: false, // disables automatic mutations' detections (advanced)
  debounceDelay: 50, // the delay on debounce used while resizing window (advanced)
  throttleDelay: 99, // the delay on throttle used while scrolling the page (advanced)
  

  // Settings that can be overridden on per-element basis, by `data-aos-*` attributes:
  offset: 120, // offset (in px) from the original trigger point
  delay: 0, // values from 0 to 3000, with step 50ms
  duration: 400, // values from 0 to 3000, with step 50ms
  easing: 'ease', // default easing for AOS animations
  once: false, // whether animation should happen only once - while scrolling down
  mirror: false, // whether elements should animate out while scrolling past them
  anchorPlacement: 'top-bottom', // defines which position of the element regarding to window should trigger the animation

});
```


### 補充
#### codepen ctrl+/取消註解
[CSS動畫](https://eyesofkids.gitbooks.io/css3/content/contents/transform3d.html)