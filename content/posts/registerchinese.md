---
title: "ASP.Net Core註冊中文編碼"
date: 2022-10-20T16:50:39+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
---
---
tags: C#
---
# ASP.Net Core 註冊中文編碼
安裝nuget套件System.Text.Encoding.CodePages
在程式內註冊
`Encoding.RegisterProvider(CodePagesEncodingProvider.Instance)`
