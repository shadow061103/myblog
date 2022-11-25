---
title: "docker的安全性"
date: 2022-10-24T15:41:29+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---
### 資源控制群組CGroup
- 可以限制資源:分配電腦資源
- 優先化:
- 用量報告:取得控制群組分配資源的統計資訊
### 核心能力機制
- 權限檢查
- 復原更改
### 資源使用限制
- 限制分配的最大記憶體
```
限制實體記憶體
-m
--memory
限制總記憶體
--memory-swap
```
- 限制CPU
```
透過權重分配
-c
--cpu-shares
--cpu-period
--cpu-quota
```
### 狀態監控
```
docker ps
看資源占有情況 --no-stream是只看當下的
docker stats --no-stream [name]
docker logs
```
