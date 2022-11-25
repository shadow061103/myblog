---
title: "Mysql的limit、offset"
date: 2022-10-27T17:08:21+08:00
draft: false
tags: ["DB"]
type: post
author: "Steve Lin"
description: ""
---
## limit [count]
 代表限制回傳幾筆
 
## limit [index,count]
index是從0開始算
limit 2,4表示從第3筆開始回傳4筆
可以想成是跳過index筆取count筆

## limit [cnt1] offset [cnt2]
跳過cnt2筆取cnt1筆
跟上面有點類似