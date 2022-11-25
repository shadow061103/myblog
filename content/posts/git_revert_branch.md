---
title: "Revert commit"
date: 2022-10-27T16:54:36+08:00
draft: false
tags: ["git"]
type: post
author: "Steve Lin"
description: ""
---

情境
不小心把uat或sit分支的東西merge到開發分支
且後來又繼續commit
可以用Revert指令還原，不建議用reset，因為就算退回去
後面開發的東西還要重做

1.首先要找到Merge的那個分支
再對他用revert 指定哪一個parent
https://medium.com/@kurosean/%E7%B5%82%E6%96%BC%E6%90%9E%E6%87%82%E5%A6%82%E4%BD%95-revert-merge-commit-a034b9e69fef