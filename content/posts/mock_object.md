---
title: "Mock測試物件"
date: 2022-10-24T15:55:06+08:00
draft: false
tags: ["單元測試"]
type: post
author: "Steve Lin"
description: ""
---
### 安裝套件
`Install-Package Moq -Version 4.18.2`
### 用法
```C#
var cal=new Mock<ICaculator>();
	cal.Setup(x=>x.Add(It.IsAny<int>(),It.IsAny<int>())).Returns(3);
	cal.Object.Add(1,2).Dump();
```
[ASP.NET MVC 單元測試系列 (3)：瞭解 Mock 假物件 ( moq )](https://blog.miniasp.com/post/2010/09/16/ASPNET-MVC-Unit-Testing-Part-03-Using-Mock-moq)
