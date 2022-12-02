---
title: "DB側錄工具&Dapper"
date: 2022-12-02T17:43:03+08:00
draft: false
tags: ["DB"]
type: post
author: "Steve Lin"
description: ""
---
# DB側錄工具

### 問題
在正式環境碰到dapper執行sql速度慢
經查有可能是 參數用匿名型別方式傳進去後被轉型(後來查到是cache到舊的sql)
int => big int
導致DB在做where查詢的時候會把欄位轉型，資料量一大就會拖慢執行速度

### 翻查dapper source code
dapper使用參數的方式有兩種
- 用匿名型別傳入
```C#
conn.QueryFirstOrDefaultAsync<decimal>(sql, new { code, startDate, endDate });
```
- 用DynamicParameters
```C#
 var param = new DynamicParameters();
param.Add("@code", storeCode, dbType: DbType.String);
param.Add("@startDate", startDate, dbType: DbType.Int32);
param.Add("@endDate", endDate, dbType: DbType.Int32);

var res = await conn.QueryFirstOrDefaultAsync<decimal>(sql, param);
```
- 最後會在背後組成一段sql 執行` sp_executesql`這個storeprecedude
- 在dapper套件內會判斷如果不是用用DynamicParameters會幫你判斷屬性型別轉成對應的sql type


![](https://imgur.com/cSYgDOh.jpg)
![](https://imgur.com/efhlygV.jpg)

[Dapper source code](https://github.com/DapperLib/Dapper/blob/main/Dapper/SqlMapper.cs)



### 查詢原因

參考[軟體主廚的文章](https://dotblogs.com.tw/supershowwei/2019/08/12/232213)
使用[XEvent 分析工具](https://learn.microsoft.com/zh-tw/sql/relational-databases/extended-events/use-the-ssms-xe-profiler?view=sql-server-ver16)或 [SQL Server Profiler](https://learn.microsoft.com/zh-tw/sql/tools/sql-server-profiler/sql-server-profiler?view=sql-server-ver16)來分析dapper產生的語法有無差異

- SQL Server Profiler目前已經被淘汰了，但如果你之前有裝過SSMS，或許電腦裡已經有安裝，可以搜尋看有沒有
- SSMS XEvent需要SSMS v17.3以上才有提供，且需要sa 完整權限才能使用

### 實際用法
因為權限問題，且只需要錄製sql語法跟看執行計畫
我是在docker建立一個暫時DB，把正式環境使用的table複製過去
然後開始XEvent側錄dapper執行的sql語法

![](https://imgur.com/0FYvpip.jpg)
畫面截圖

### 參考
[料理佳餚Dapper用起來很友善，但是預設的參數型別對執行計劃不太友善](https://dotblogs.com.tw/supershowwei/2019/08/12/232213)

[SQL Server Hidden “Load Evil” (Performance Issue)With Dapper | by Jithil Jishad | Medium](https://jithilmt.medium.com/sql-server-hidden-load-evil-performance-issue-with-dapper-465a08f922f6)

[資料類型轉換 (資料庫引擎) - SQL Server | Microsoft Learn](https://learn.microsoft.com/zh-tw/sql/t-sql/data-types/data-type-conversion-database-engine?view=sql-server-ver16)

[SQL语句在查询分析器执行很快，程序 Dapper 参数化查询就很慢（parameter-sniffing](https://www.cnblogs.com/Irving/p/3951220.html)