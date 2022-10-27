---
title: "git merge 不直接合併的方法"
date: 2022-10-27T16:53:23+08:00
draft: false
tags: ["git"]
type: post
author: "Steve Lin"
description: ""
---
---
tags: Git
---
# git merge 不直接合併的方法
- ` git merge member --no-commit --no-ff`
    -  --no-commit就是不產生一個commit,資料會放到暫存區
    -  --no-ff不要fast forward
- 不commit的話可以`git reset --merge`或`git merge --abort`