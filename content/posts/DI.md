---
title: "DI注入不同實體做法"
date: 2022-10-20T17:03:53+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "DI注入不同實體做法"
---

# .Net core DI注入不同實體做法
### 註冊DI使用Func
- 有兩個不同class都繼承IESCaseRepository
- 使用EnvTag做區別
```C#
services.AddScoped<ESCaseRepository>();
services.AddScoped<ESCaseCloudRepository>();

services.AddScoped<Func<EnvTag, IESCaseRepository>>(sp => env =>
            {
                return env switch
                {
                    EnvTag.Cloud => sp.GetService<ESCaseCloudRepository>(),
                    EnvTag.Local => sp.GetService<ESCaseRepository>(),
                    _ => throw new Exception($"找不到 {EnvTag.Cloud}的注入設定，請在DI設定")
                };
            });
```
### 使用時
- 記得宣告
```C#
private IESCaseRepository _esWebCaseRepository;

private IESCaseRepository _esCloudWebCaseRepository;
```
- 在建構試寫Func<EnvTag, IESCaseRepository> resolver
- 使用時像這樣
```C#
 _esWebCaseRepository = resolver(EnvTag.Local);
_esCloudWebCaseRepository = resolver(EnvTag.Cloud);
```

### 兩個class邏輯一樣
- 只有差在裡面的建構式用不同實體
- 可以不用寫兩個class，只需要一個然後用func去指定實做就好
- 注入 ElasticClient是實作內容不一樣
```C#
 services.AddScoped<Func<EnvTag, IESCaseRepository>>(sp => env =>
            {
                var resolver = sp.GetService<Func<EnvTag, ElasticClient>>();
                return new ESCaseRepository(env, resolver, sp.GetService<ICommonGeographyRepository>());
            });
```