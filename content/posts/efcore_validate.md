---
title: "EFcore模型驗證預設不檢查插入問題"
date: 2022-10-20T16:44:27+08:00
draft: false
tags: ["Csharp","efcore"]
type: post
author: "Steve Lin"
description: ""
slug: "EFcore模型驗證預設不檢查插入問題"
---
### 情境
在model上的屬性設置如下
```C#
[Column("bank_trade_no")]
[MaxLength(30)]
[Required]
public string BankTradeNo { get; set; }

```
DB欄位是設定成可為null且預設Null的值
### 問題發生原因
問題發生在 DB有一筆資料的欄位是null，用ef core撈出來的時候報exception
error stack
```
System.InvalidOperationException: An error occurred while reading a database value for property 'AclinkTradeRecord.BankTradeNo'. The expected type was 'System.String' but the actual value was null.
 ---> System.InvalidCastException: Unable to cast object of type 'System.DBNull' to type 'System.String'.
   at MySqlConnector.Core.Row.GetString(Int32 ordinal) in /_/src/MySqlConnector/Core/Row.cs:line 371
   at MySqlConnector.MySqlDataReader.GetString(Int32 ordinal) in /_/src/MySqlConnector/MySqlDataReader.cs:line 282
   at lambda_method11(Closure , QueryContext , DbDataReader , ResultContext , SingleQueryResultCoordinator )
   --- End of inner exception stack trace ---
   at lambda_method11(Closure , QueryContext , DbDataReader , ResultContext , SingleQueryResultCoordinator )
   at Microsoft.EntityFrameworkCore.Query.Internal.SingleQueryingEnumerable`1.AsyncEnumerator.MoveNextAsync()
   at Microsoft.EntityFrameworkCore.Query.ShapedQueryCompilingExpressionVisitor.SingleOrDefaultAsync[TSource](IAsyncEnumerable`1 asyncEnumerable, CancellationToken cancellationToken)
   at Microsoft.EntityFrameworkCore.Query.ShapedQueryCompilingExpressionVisitor.SingleOrDefaultAsync[TSource](IAsyncEnumerable`1 asyncEnumerable, CancellationToken cancellationToken)
   at WebApplication2.RepositoryBase.GetFirstOrDefaltAsync[TEntity](Expression`1 predicate) in C:\SideProject\WebApplication2\WebApplication2\IRepository.cs:line 40
   at WebApplication2.TradeRecordRepository.GetTradeAsync(Expression`1 predicate) in C:\SideProject\WebApplication2\WebApplication2\IRepository.cs:line 105
   at WebApplication2.Controllers.WeatherForecastController.GetAsync() in C:\SideProject\WebApplication2\WebApplication2\Controllers\WeatherForecastController.cs:line 67
   at Microsoft.AspNetCore.Mvc.Infrastructure.ActionMethodExecutor.TaskOfIActionResultExecutor.Execute(IActionResultTypeMapper mapper, ObjectMethodExecutor executor, Object controller, Object[] arguments)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeActionMethodAsync>g__Awaited|12_0(ControllerActionInvoker invoker, ValueTask`1 actionResultValueTask)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeNextActionFilterAsync>g__Awaited|10_0(ControllerActionInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Rethrow(ActionExecutedContextSealed context)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeInnerFilterAsync>g__Awaited|13_0(ControllerActionInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|19_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
   at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
   at Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
   at Swashbuckle.AspNetCore.SwaggerUI.SwaggerUIMiddleware.Invoke(HttpContext httpContext)
   at Swashbuckle.AspNetCore.Swagger.SwaggerMiddleware.Invoke(HttpContext httpContext, ISwaggerProvider swaggerProvider)
   at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)
```
### 解決方式
1.思考為何有設定[Require]卻可以插入null值進DB?
後來有查到ef core現在`SaveChangesAsync()`方法不會做模型驗證了
需要自己override方法
```C#
//加在context上
public  async Task<int> SaveChangesAsync()
        {
            var changedEntities = ChangeTracker
                .Entries()
                .Where(_ => _.State == EntityState.Added ||
                            _.State == EntityState.Modified);

            var errors = new List<ValidationResult>(); // all errors are here
            foreach (var e in changedEntities)
            {
                var vc = new ValidationContext(e.Entity, null, null);
                Validator.TryValidateObject(
                    e.Entity, vc, errors, validateAllProperties: true);
            }

            return await base.SaveChangesAsync();
        }
```
2.null值撈出來會出錯問題
這部分就要調整model跟DFB一致，看要都是可以null或是not null
[EF Core 已經不會在 SaveChanges() 的時候對實體模型進行額外驗證(Will)](https://blog.miniasp.com/post/2022/04/23/EF-Core-has-no-ValidateOnSaveEnabled-anymore)
[參考解法](https://www.bricelam.net/2016/12/13/validation-in-efcore.html)
[自訂驗證-余小章](https://dotblogs.com.tw/yc421206/2018/12/16/model_validation)