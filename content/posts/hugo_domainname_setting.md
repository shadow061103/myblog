---
title: "hugo使用github page+Gandi架設"
date: 2022-12-16T17:25:40+08:00
draft: true
tags: ["hugo"]
type: post
author: "Steve Lin"
description: ""
---


### 需求
想使用自己的網域套用到github page預設的網域上
- 原先:< account >.github.io
- 改成 < account >.site

### 註冊域名
- 域名可以透過域名購買商購買
    - [Gandi](https://www.gandi.net/en)
    - [google domain](https://domains.google/) 
- 因為我是買Gandi的，所以用這個來講解
1.這邊可以輸入你要的名稱，會列出所有可能的domain名稱跟價格
![](https://miro.medium.com/max/1400/0*57a9XnpMkfm00MkS.webp)

2.購買後點進去DNS Records設定
![](https://imgur.com/YQ60scC.jpg)
Type A設定github的IP
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
CName設定:注意hostname後面要加上`.`，是用來連結到你的github page
![](https://imgur.com/LLHTr1L.jpg)

### 設定github domain
位置:repository > setting > Custom domain
把剛註冊好的域名貼上去
![](https://imgur.com/XQoFjvA.jpg)

### hugo設定調整
config.toml的baseUrl記得要改成新的域名

### 什麼是DNS?
- DNS (Domain Name Server):翻成中文是網域名稱伺服器，可以理解為「負責處理域名的伺服器」
- DNS幫我們做了一些事情，DNS 就像生活中的地政事務所這樣的管理單位，透過幫每個 IP 位置取個名字，把這些原本只有電腦能夠理解的位置數據，變成人類也很好記憶、理解的名稱
### DNS類型
- A Record:A 代表 Address，也就是紀錄IP Address 與網域名稱的配對，這個紀錄是在設定Domain Name時最重要的項目
- CNAME ： CNAME 是網域名稱的別名，用來將子域名指向另外一個主機的域名，最常見的就是將 www.abc.com (子域名） 指向 abd.com（購買的主域名） ，避免使用者因為意外而找不到網站（夠買主域名後，就可以從域名商的設定後台，新增子域名，是不需要另外付費的）
- ALIAS : 與 CNAME 記錄很相像，差別在 CNAME 別名只能用於子域名，而 ALIAS 記錄能夠用在主域名，但這麼做會影響對主域名的域名解析，所以不能與 A 紀錄同時使用
### 參考
[Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)
[個人技術站一把罩！部落格建置大全（二）- 將 Github Page 串上自己的域名](https://medium.com/%E5%89%8D%E7%AB%AF%E5%AF%A6%E5%8A%9B%E4%B8%89%E6%98%8E%E6%B2%BB/%E5%80%8B%E4%BA%BA%E6%8A%80%E8%A1%93%E7%AB%99%E4%B8%80%E6%8A%8A%E7%BD%A9-%E9%83%A8%E8%90%BD%E6%A0%BC%E5%BB%BA%E7%BD%AE%E5%A4%A7%E5%85%A8-%E4%BA%8C-%E5%B0%87-github-page-%E4%B8%B2%E4%B8%8A%E8%87%AA%E5%B7%B1%E7%9A%84%E5%9F%9F%E5%90%8D-8f7e11cf2687)
[什麼是DNS？DNS紀錄怎麼設定？](https://asper.tw/dns/)