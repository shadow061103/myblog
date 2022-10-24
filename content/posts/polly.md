---
title: "Polly錯誤自動重試機制"
date: 2022-10-20T16:54:03+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: "Polly錯誤自動重試機制"
slug: ""
---
---
tags: C#
---
# Polly錯誤自動重試機制
### 如何使用
- 需要先安裝nuget套件 (Polly)
- 主要結構
```C#
 Policy.Handle<HttpRequestException>()
                .OrResult<HttpResponseMessage>(result => result.StatusCode != HttpStatusCode.OK)
                .Retry(3, onRetry: (exception, retryCnt) =>
                {
                    Console.WriteLine($"polly 呼叫api異常 進行第{retryCnt}次重試,error{exception.Result.StatusCode}");
                }).Execute(DoMockRequest);
```
### Polly支援的策略
- `Handle<T>`:故障時的處理機制，或是指定要處理什麼樣的異常,有多個的話可以用`Or<T>`
- `HandleResult<T>`:依據回傳內容進行故障處理，有多個可以用OrResult
- Retry:重試機制，定義發生故障時要重試的次數或指定工作
- WaitAndRetry:可以定義間隔秒數重試
```
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(5, retryAttempt => 
 TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)) 
  );
```
- Execute:要執行的任務
- Fallback (替代措施):出錯時啟用備援方案，維持營運
- Circuit Breaker(熔斷):一定期間錯誤次數超過上限，即先停止執行相關動作
- Timeout:超過一定時間就放棄
- Bulkhead Isolation(艙倉隔離):避免出錯請求耗用過多資源拖垮整個系統，限定作業可用資源上限(主要是限制同時執行的請求數量)，隔離其對其他系統的影響
- PolicyWrap:組合上述多種措施混用，彈性因應

### 範例
把實作方法先包成Action方法，這樣就可以比較彈性
或是直接在ExecuteAsync呼叫寫好的方法
```C#
private static async Task Main(string[] args)
        {
            var test = new Test();

            Func<Task<bool>> action = async () =>
            {
                Random myObject = new Random();
                var num = myObject.Next(2, 5);
                var res = await test.Check(num);
                Console.WriteLine(res);
                return res;
            };
            var policy = Policy.Handle<Exception>()
                .WaitAndRetryAsync(new[]
                                    {
                                        TimeSpan.FromSeconds(2),
                                        TimeSpan.FromSeconds(3),
                                        TimeSpan.FromSeconds(4)
                                    }, (reponse, ts) =>
                                    {
                                        Console.WriteLine($"間隔{ts.TotalSeconds}秒 繳費退款");
                                    });

            try
            {
                await policy.ExecuteAsync(action);
            }
            catch (Exception ex)
            {
                Console.WriteLine("error");
            }

            Console.ReadKey();
        }
    }

    public class Test
    {
        public async Task<bool> Check(int number)
        {
            Console.WriteLine($"input {number}");
            if (number == 3)
                throw new Exception("boomb!");

            if (number < 2)
            {
                return await Task.FromResult(false);
            }

            for (var divisor = 2; divisor <= Math.Sqrt(number); divisor++)
            {
                if (number % divisor == 0)
                {
                    return await Task.FromResult(false);
                }
            }
            return await Task.FromResult(true);
        }
    }
```

### 資料來源
[[NETCore] 使用 Polly 實現重試 (Retry) 策略](https://marcus116.blogspot.com/2019/06/netcore-polly-retry.html)
https://github.com/App-vNext/Polly
https://github.com/App-vNext/Polly/wiki
[黑暗執行緒](https://blog.darkthread.net/blog/polly/)
[使用 IHttpClientFactory 和 Polly 原則實作指數輪詢的 HTTP 呼叫重試](https://docs.microsoft.com/zh-tw/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)