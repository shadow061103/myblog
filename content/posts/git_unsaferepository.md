---
title: "git unsafe repository錯誤解決"
date: 2022-10-27T16:51:00+08:00
draft: true
tags: ["git"]
type: post
author: "Steve Lin"
description: ""
---


# git unsafe repository錯誤解決

在下git指令的時候會跳出錯誤
![](https://imgur.com/N8474PA.jpg)
原因是git更新的安全修正
要解決可以用它上面的指令
或是去找gitconfig(路徑:C:/[user]\[user name]\.gitconfig)
加上這一段
```
[safe]
	directory = *
```


[Git unsafe repository 錯誤與 Gitea 整合問題](https://blog.darkthread.net/blog/git-unsafe-repo-n-gitea/)