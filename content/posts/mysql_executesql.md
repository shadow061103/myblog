---
title: "EF Core 5的ExecuteSqlRaw的雷"
date: 2022-11-25T15:46:02+08:00
draft: true
tags: ["DB"]
type: post
author: "Steve Lin"
description: ""
---
在程式裡的寫法是這樣
```C#
var paramItems = new[]{
                new MySqlParameter("@ids", ids)
            };

var sql = @"update table set invoice_status=1 where id in @ids;";

await _context.Database.ExecuteSqlRawAsync(sql, paramItems);
```
會拋錯誤因為MySqlParameter不支援傳list的資料

[文件說明](https://mysqlconnector.net/troubleshooting/parameter-types/)

錯誤訊息
```
Parameter type SelectListIterator`2 is not supported; see https://fl.vu/mysql-param-type. Value: System.Linq.Enumerable+SelectListIterator`2[PXPlus.Invoice.Domain.EPIPTransferTradeAgg.EPIPTransferTrade,System.Int64]
```

### 查到的解法
- 使用mysql的FIND_IN_SET語法

https://stackoverflow.com/questions/5681320/add-listint-to-a-mysql-parameter
```C#
 using (var context = new EfcoretestContext())
                {
                    var trades = await context.EPIPTransferTrade.ToListAsync();

                    var sql = @"update table set invoice_status = 1 where FIND_IN_SET(id,@ids);";

                    var ids = trades.Select(c => c.Id);
                    var paras = new[] {
                    new MySqlParameter("@ids",string.Join(",",ids))};
                  
                    await context.Database.ExecuteSqlRawAsync(sql, paras);
```
- 直接用字串插值也可以，前提是傳入的參數不能是使用者輸入的，比方說是列表取出的欄位

### 文件
[ExecuteSqlRaw](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.executesqlraw?view=efcore-5.0#microsoft-entityframeworkcore-relationaldatabasefacadeextensions-executesqlraw(microsoft-entityframeworkcore-infrastructure-databasefacade-system-string-system-collections-generic-ienumerable((system-object))))
