---
title: "DotNet Core的Redis"
date: 2022-12-19T15:12:10+08:00
draft: false
tags: ["redis"]
type: post
author: "Steve Lin"
description: ""
---
---
tags: Redis
---
# .Net Core的Redis
### 套件
- `StackExchange.Redis`,
- `Microsoft.Extensions.Caching.StackExchangeRedis`
### DI注入
- 這邊用docker起的redis
- 有包含 .net core提供的分散式快取，跟套件的多路複用用法
```C#
/// <summary>
    /// Redis連線
    /// </summary>
    public static class RedisCacheServiceCollectionExtensions
    {
        public static IServiceCollection AddRedisCache(this IServiceCollection services)
        {
            services.AddSingleton<IConnectionMultiplexer>(sp =>
            {
                return ConnectionMultiplexer.Connect(services.GetGetConfigString());
            });

            //distributed cache 以下兩種擇一使用
            //方法一
            services.AddOptions<RedisCacheOptions>()
                .Configure<IServiceProvider>((options, sp) =>
                {
                    options.Configuration = services.GetGetConfigString();
                    options.InstanceName = "";
                });

            services.AddSingleton<IDistributedCache, RedisCache>();

            //方法二
            //services.AddStackExchangeRedisCache(options =>
            //{
            //    options.Configuration = services.GetGetConfigString();
            //    options.InstanceName = "";
            //});

            return services;
        }

        private static string GetGetConfigString(this IServiceCollection services)
        {
            return "127.0.0.1:6379,syncTimeout=8000";
        }
    }
```
### IDistributedCache 
- 有提供幾個方法使用
    - GetAsync:接受字串索引鍵，並以陣列形式抓取快取的專案
    - SetAsync:使用字串索引鍵，將 (為 byte[] 陣列) 的專案加入至快取
    - RefreshAsync:reset逾時時間
    - RemoveAsync:會根據其字串索引鍵來移除快取專案
- 分散式快取顧名思義，快取是指向外部空間，所以Server重啟後快取還會存在
- 只能使用byte[]，可以用MessagePack
![](https://blog.johnwu.cc/images/i20-1.png)

### IDistributedCache創建的類型
- 儲存資料是用Hash
- 但是自己加了很多key上去，比較重要的就是`data`，主要透過這個來取值
- [RedisCache.cs](https://github.com/aspnet/Caching/blob/master/src/Microsoft.Extensions.Caching.StackExchangeRedis/RedisCache.cs)
- 內部實作IDistributedCache也是用lua script的方式

[範例專案](https://github.com/shadow061103/RedisShakeHand)