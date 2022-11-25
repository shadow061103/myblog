---
title: "EFCore使用"
date: 2022-10-20T16:55:56+08:00
draft: false
tags: ["Csharp","efcore"]
type: post
author: "Steve Lin"
description: ""
slug: "EFCore使用"
---
## 安裝
- 下載nuget
```
cli
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
套件管理員主控台
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```
- 非必要 建資料庫
```
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design

dotnet ef migrations add InitialCreate
dotnet ef database update

這邊用法
//要指定資料夾
dotnet ef migrations add addPayToken -o Infrastructure/Migrations
dotnet ef database update
```
這邊如果要新增database
需要先寫好entity跟context
```
產生Script
dotnet ef migrations script --context FinancialContext 
```
- 可以使用EF core power tool建立DBContext檔
### 使用
- 如果只要單一實例DBContext，可以在startUp做DI設定，使用時在建構子注入
```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddDbContext<ApplicationDbContext>(
        options => options.UseSqlServer("name=ConnectionStrings:DefaultConnection"));
}
//DBContext要使用這個函式
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
}
```
- new 初始化，覆寫 OnConfiguring 方法
```C#
public class ApplicationDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test");
    }
}
```

- 可以使用DbContextOptionsBuilder來建立 DbContextOptions 物件
```C#
var contextOptions = new DbContextOptionsBuilder<ApplicationDbContext>()
    .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test")
    .Options;

using var context = new ApplicationDbContext(contextOptions);
```


### Tracking VS No Tracking
- 用tracking的查詢方式，ef core會偵測所做的變更，呼叫SaveChanges 或 SaveChangesAsync的時候寫進資料庫
- 使用之後要釋放掉

### User Secret
儲存敏感性資料 EX資料庫帳密
[User Secret](https://docs.microsoft.com/zh-tw/aspnet/core/security/app-secrets?view=aspnetcore-5.0&tabs=windows)
