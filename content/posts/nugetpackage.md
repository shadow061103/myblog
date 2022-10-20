---
title: "打包Nuget套件"
date: 2022-10-20T17:03:25+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "打包Nuget套件"
---

# 打包Nuget套件
### 設定練習用套件來源
- 公司練習的位置是http://package.evertrust.com.tw/feeds/Exercise


### 開發環境設定
- 安裝node.js 
    - https://nodejs.org/en/ 下載安裝檔案
- 設定npm
    -  `npm config set registry="http://package.evertrust.com.tw/npm/registry`
    -  `npm config set strict-ssl=false`
- 安裝nuget命令列
    - `npm install nuget-cli -g`
- 下載nuget.exe
    - [download](https://www.nuget.org/downloads) 
    - 把nuget.exe.檔案路徑加到環境變數PATH，重開cmd就可以用了

## 使用NuGet Package Explorer製作套件
- 下載並開啟[NuGet Package Explorer](https://www.microsoft.com/zh-tw/p/nuget-package-explorer/9wzdncrdmdm3?activetab=pivot:overviewtab)
- Create a new package
![](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer/raw/main/images/screenshots/CommonTasks.png)
- 編輯Package metadata(id、version、description...)
![](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer/raw/main/images/screenshots/EditMetadata.png)
```
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
  <metadata>
    <id>EvertrustStrongLibrary-g5341</id>
    <version>1.0.0</version>
    <title>EvertrustStrongLibrary-g5341</title>
    <authors>G5341</authors>
    <owners>Evertrust</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>My package description.</description>
  </metadata>
</package>
```
- Add Lib Folder新增資料夾
- 因為專案版本是 .net standard所以增加一個netstandard2.0資料夾
- 把專案編譯好的dll放進netstandard2.0資料夾中
- 按儲存 檔名會是{Id}.{Version}.nupkg 
    - `EvertrustStrongLibrary-g5341.1.0.0.nupkg`
- 最後上傳package server
    - 打開 http://package.evertrust.com.tw/feeds/Exercise
    - add package


## 使用command製作套件
- 打開套件管理主控台或在專案目錄下打開cmd，把nuget更新到最新版本，下指令 `nuget update -self` 
- 切到專案目錄下，建立nuspec `nuget spec`，再把檔案加到專案中，記得要先在專案裡面調整metadata不然NuGet Package Explorer打不開
- 使用Nuget Package Explorer編輯剛建好的nuspec
    - 編輯Metadata
    - 清除所有content
    - 用save Matadata As存檔，檔名選原來那個覆蓋過去`EvertrustStrongLibrary.nuspec`
- 接著在.csproj的資料夾下指令
    - 封裝`nuget pack .\EvertrustStrongLibrary.csproj`
    - 會產出'EvertrustStrongLibrary-g5341.1.0.1.nupkg'
- 封裝&上傳`nuget push EvertrustStrongLibrary-g1359.1.0.1.nupkg ffPrdXiAi22vjcBrRhQ4NOVuFDkxGtUd -source http://package.evertrust.com.tw/nuget/Exercise`

### 封裝比較
- Nuget Package Explorer
    - 簡單易用
    - 版本更新無版控，無法整合CI
- Command
    - 門檻高
    - 有版控，整合CI
### 上傳比較
- Web上傳
    - 簡單易用
- Command
    - 門檻高 

### 發行套件到 [nuget.org](https://www.nuget.org/)
- [微軟官方文件](https://docs.microsoft.com/zh-tw/nuget/nuget-org/publish-a-package)
- .net standard的專案，專案屬性套件設定好版本號之後，要勾選*在建置時產生nuget套件*，產生出要推送用的.nupkg檔案
- 要注意套件識別碼不能跟預設命名空間一樣，不然會出現`package id is not available`的錯誤
- package推上去之後要等幾分鐘才會出現在列表，因為要進行檢查跟編制索引
    - [說明](https://docs.microsoft.com/en-us/nuget/nuget-org/publish-a-package#package-validation-and-indexing)
- 可以在`nuget.org` 點upload上傳nupkg檔案(要注意不能刪除 只能選擇不能被搜尋 [刪除政策](https://docs.microsoft.com/zh-tw/nuget/nuget-org/policies/deleting-packages)) 
#### 指令
- dotnet nuget publish 
    - `dotnet nuget push YourPackage.nupkg --api-key <your_API_key> --source https://api.nuget.org/v3/index.json`
- 使用nuget push發行
    - `nuget setApiKey <your_API_key>` 之後不用再設定
    - `nuget push YourPackage.nupkg -Source https://api.nuget.org/v3/index.json`


