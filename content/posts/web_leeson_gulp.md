---
title: "環境安裝流程"
date: 2022-10-27T17:12:31+08:00
draft: false
tags: ["網頁切版直播課程"]
type: post
author: "Steve Lin"
description: ""
---

# gulp 環境安裝流程

#### 1.安裝[Node.js](https://nodejs.org/en/)，打開終端機或命令提示字元，輸入`node -v`，安裝完成會顯示版本號

#### 2.在終端機安裝gulp，繼續輸入指令 `npm i gulp@4 -g` ,輸入`gulp -v`查詢是否有回傳版本號

#### 3.下載此[資料夾](https://github.com/hexschool/web-layout-training-gulp)並解壓縮

#### 4.移動到該資料夾 

#### 5.輸入指令 `npm install` 安裝插件，這是安裝專案內package.json列出的需要元件
#### `npm install gulp -g` 會在全域下安裝gulp 

#### 6.輸入指令`gulp`執行

#### 7.若步驟6執行成功，會打開瀏覽器，看到指定畫面

## 指令列表
- gulp - 執行開發模式(會開啟模擬瀏覽器並監聽相關檔案)
- gulp build - 執行編譯模式(不會開啟瀏覽器)
- gulp clean - 清除 dist 資料夾
- gulp deploy - 將 dist 資料夾部署至 GitHub Pages
- npm install gulp-cli -g 才能使用gulp指令




