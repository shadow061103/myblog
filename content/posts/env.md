---
title: "取得.net環境變數"
date: 2022-10-20T16:52:39+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
---
- 最新版的環境變數有分兩個`DOTNET_ENVIRONMENT`跟`ASPNETCORE_ENVIRONMENT`
- 可以在方法注入`IHostEnvironment`
- 然後用_hostEnvironment.IsProduction()判斷