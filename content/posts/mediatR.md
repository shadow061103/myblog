---
title: "MediatR"
date: 2022-10-20T17:01:00+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "MediatR"
---

# MediatR

### 安裝套件
```
Install-Package MediatR
Install-Package MediatR.Extensions.Microsoft.DependencyInjection
```
### DI注入
```
public void ConfigureServices(IServiceCollection services)
{
    services.AddMediatR(Assembly.GetExecutingAssembly());
    //IRequestPreProcessor跟IRequestPostProcessor都不用再注入
    //但是IPipelineBehavior要
}
```
[git](https://github.com/jbogard/MediatR/tree/master/src/MediatR)
### IRequestHandler
- [IRequestHandler<in TRequest, TResponse>](https://github.com/jbogard/MediatR/blob/master/src/MediatR/IRequestHandler.cs)
- 如果是沒有回傳值TResponse可以放Unit
- **IRequest**可不放(不回傳)或寫要回傳的型別`IRequest<T>`
- 只能一對一執行handler



### INotificationHandler
- [INotificationHandler<in TNotification, TResponse>](https://github.com/jbogard/MediatR/blob/master/src/MediatR/INotificationHandler.cs)
- T要繼承**INotification**，沒有回傳值
- 是一對多的事件(event)
- 要用publish通知執行動作

### 實作
[看git比較快](https://github.com/shadow061103/MediatRSample)

### Pipeline
- `IRequestPreProcessor<TRequest>`是在request之前執行
- `IRequestPostProcessor<TRequest, TResponse>`是在request之後執行，所以可以拿到response
- `IPipelineBehavior`在執行方法前後執行
- 執行順序:IRequestPreProcessor>IPipelineBehavior>方法>IPipelineBehavior>IRequestPostProcessor
- 
