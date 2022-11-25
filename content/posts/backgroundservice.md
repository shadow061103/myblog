---
title: "背景執行工作"
date: 2022-10-20T16:59:46+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: ""
---

- 要執行的工作繼承BackgroundService
- 實作`Task ExecuteAsync(CancellationToken stoppingToken)`
- 設定時間`await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);`
### StartUp.cs註冊
- `services.AddHostedService<CacheCheckService>();`
- 其他相關連有用到的都要改成Singleton，但也可以不用，需要使用`IServiceScopeFactory`
```C#
public class CacheCheckService
{
	//宣告
	private readonly IServiceScopeFactory _serviceScopeFactory;
	public CacheCheckService(IServiceScopeFactory serviceScopeFactory)
	{
		_serviceScopeFactory = serviceScopeFactory;
	}
	//會用到的service
	private IChatNotifyService _notifyService;

	private IChatNotifyService NotifyService
	{
		get
		{
			if (this._notifyService != null)
			{
				return this._notifyService;
			}

			var scope = this._serviceScopeFactory.CreateScope();
			var notifyService = scope.ServiceProvider.GetRequiredService<IChatNotifyService>();
			this._notifyService = notifyService;
			return this._notifyService;
		}
	}

}
```

### 微軟文件
[在 ASP.NET Core 中使用託管服務的背景工作](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-3.1&tabs=visual-studio)
[在微服務中使用 IHostedService 和 BackgroundService 類別實作背景工作](https://docs.microsoft.com/zh-tw/dotnet/architecture/microservices/multi-container-microservice-net-applications/background-tasks-with-ihostedservice)