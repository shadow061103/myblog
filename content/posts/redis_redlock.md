---
title: "Redis分布式鎖"
date: 2022-12-19T15:38:26+08:00
draft: true
tags: ["redis"]
type: post
author: "Steve Lin"
description: ""
---


### 安裝套件
StackExchange.Redis
RedLock.net

### 用法
- 自己寫一個Factory來管理連線
AbortOnConnectFail =false表示連線失敗就不重試，會直接噴錯

```C#
public interface IRedisConnectionFactory
    {
        RedLockFactory GetRedLockFactory();
    }
public class RedisConnectionFactory : IRedisConnectionFactory
    {
        private Lazy<ConnectionMultiplexer> Connection { get; set; }

        public RedisConnectionFactory(string redisUrl)
        {
            var connectionString = redisUrl;
            var options = ConfigurationOptions.Parse(connectionString);
            //斷線是否要重試
            options.AbortOnConnectFail = true;
            Connection = new Lazy<ConnectionMultiplexer>(() => ConnectionMultiplexer.Connect(options));
        }

        public RedLockFactory GetRedLockFactory()
        {
            var multiplexers = new List<RedLockMultiplexer> { Connection.Value };
            return RedLockFactory.Create(multiplexers);
        }
    }
```
- DI注入
考慮到redis斷線問題，用Scope注入就算斷線，每次去取都會重新建立實例
用Singleton的話，只有一開始會去連線，後面只要一次斷線，之後都會拋Exception
[參考](https://gist.github.com/JonCole/36ba6f60c274e89014dd)
```C#

private void AddRedis(IServiceCollection services)
        {
            services.AddScoped<IRedisConnectionFactory>(s =>
            {
                return new RedisConnectionFactory(Configuration.GetSection("RedisSettings:Url").Value);
            });
        }
```

- 把方法包起來
這邊我是用scrutor註冊一個Decorate的Service，把原來的方法整個包起來
[使用 RedLock.net 搭配 redis 達成分散式 Lock](https://blog.yowko.com/redlocknet-redis-lock/)
```C#
 var redisLockFactory = _redisConnectionFactory.GetRedLockFactory();
                var lockKey = $"lockkey";
                var expiry = TimeSpan.FromSeconds(30);
                using (var redLock = await redisLockFactory.CreateLockAsync(lockKey, expiry))
                {
                    if (redLock.IsAcquired)
                    {
                        return await _appService.ExecuteAsync(appropriationEventDTO);
                    }
                    else
                    {
                       //取不到鎖要做的處理
                    }
                }
```
### 可能會有的問題
如果redis是master/slave的架構
在master拿到鎖之後，如果shut down掉
但因為master還沒複製到slave
slave升成master之後，別的process還是可以拿到鎖
等於是有兩個process同時拿到鎖，直到時間到期釋放掉
但發生機率應該不大


參考文章

https://isdaniel.github.io/redlock-deepknow/
https://gist.github.com/JonCole/36ba6f60c274e89014dd