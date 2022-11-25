---
title: "取得遠端分支"
date: 2022-10-27T16:52:43+08:00
draft: false
tags: ["git"]
type: post
author: "Steve Lin"
description: ""
---
### 1. Fetch all remote branches
```
git fetch origin
```
### 2.List the branches available for checkout
```
git branch -a
```

### Pull changes from a remote branch
```
git checkout -b fix-failing-tests origin/fix-failing-tests
```

### 另一種做法
```
取得遠端分支
git branch --remote
拉到本地端新建一個追蹤分支
git checkout -t origin/refactoring

或是直接把origin拿掉
git checkout refactoring

```