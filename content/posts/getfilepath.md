---
title: "取檔案路徑作法"
date: 2022-10-20T17:05:35+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "取檔案路徑作法"
---
- AppDomain.CurrentDomain.BaseDirectory 執行的應用程式在哪裡
> D:\CorporateProject\Yungching.Webservice.Rent\Yungching.Webservice.Rent\bin\Debug\netcoreapp3.1\
- Assembly.GetExecutingAssembly().Location 目前執行的組件在哪裡
> D:\CorporateProject\Yungching.Webservice.Rent\Yungching.Webservice.Rent\bin\Debug\netcoreapp3.1\Yungching.Webservice.Rent.Service.dll
- Directory.GetCurrentDirectory() 在哪裡下執行指令
> D:\CorporateProject\Yungching.Webservice.Rent\Yungching.Webservice.Rent
- 同第一種
```C#
var path = new Uri(Assembly.GetExecutingAssembly().CodeBase).AbsolutePath;
var dir = Path.GetDirectoryName(path);
```
