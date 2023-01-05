---
title: "dotnet-dump analyze分析"
date: 2023-01-05T14:06:47+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
---

## 測試步驟(使用微軟專案實現要測試情境)
### 建立docker container
[範例專案](https://github.com/dotnet/samples/tree/main/core/diagnostics/DiagnosticScenarios)

- 用svn把專案clone下來
`svn co https://github.com/dotnet/samples/trunk/core/diagnostics/DiagnosticScenarios`
- 先建置再release
```
dotnet clean
dotnet restore
dotnet build -c Release
```
範例專案可能需要調整csproj檔案如下，原本的版本號會build不過
```
<PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" Version="6.0.1" />
  </ItemGroup>
```
- 新增Dockerfile在專案目錄
```
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

COPY  bin/Release/net6.0 .

ENTRYPOINT ["dotnet", "DiagnosticScenarios.dll"]
```

- 建docker image
`docker build -t dumptest .`
- run container
`docker run --name diagnostic --privileged=true -p 888:80 -d dumptest`
privileged是要給容器許可權
[docker privileged](https://docs.docker.com/engine/reference/commandline/run/#full-container-capabilities---privileged)

- 連網址檢查container有沒有成功
http://localhost:888/api/values
### 生成dump檔案
- 用專案提供的api測試鎖死跟記憶體耗用問題
http://localhost:888/api/diagscenario/deadlock
http://localhost:888/api/diagscenario/memleak/101024
- 建立dump檔案
透過find找到netcore自帶的createdump工具；
執行createdump路徑 PID命令建立dump檔案
(如果容器內只有一個應用，一般PID預設為1，也可以使用top命令來檢視PID)
找到之後就退出contaner
```
PS C:\Users\steve> docker exec -it diagnostic bash
root@b4a4d3879eb0:/app# find / -name createdump
/usr/share/dotnet/shared/Microsoft.NETCore.App/6.0.12/createdump
root@b4a4d3879eb0:/app# /usr/share/dotnet/shared/Microsoft.NETCore.App/6.0.12/createdump 1
[createdump] Gathering state for process 1 dotnet
[createdump] Writing minidump with heap to file /tmp/coredump.1
[createdump] Written 2909171712 bytes (710247 pages) to core file
[createdump] Target process is alive
[createdump] Dump successfully written
```
- 把coredump.1檔案copy出來到本地資料夾
```
docker cp b4a4d3879eb0:/tmp/coredump.1 C:\github\DiagnosticScenarios
```

### 分析dump檔案
dotnet-dump工具依賴dotnet的sdk，如果本機安裝了sdk可以直接在宿主機中分析
或是可以pull一個sdk的image，建立一個臨時環境用於分析
這邊使用第二種作法


- 建立測試container，要把dump檔案copy進去，或是用volume掛載資料夾
`docker run --rm -it -v C:/github/tmp:/tmp/coredump mcr.microsoft.com/dotnet/sdk:6.0`
- 建好容器之後須安裝dotnet-dump工具
`dotnet tool install -g dotnet-dump`
```
root@e881fccccede:/# dotnet tool install -g dotnet-dump
Tools directory '/root/.dotnet/tools' is not currently on the PATH environment variable.
If you are using bash, you can add it to your profile by running the following command:

cat << \EOF >> ~/.bash_profile
# Add .NET Core SDK tools
export PATH="$PATH:/root/.dotnet/tools"
EOF

You can add it to the current session by running the following command:

export PATH="$PATH:/root/.dotnet/tools"

You can invoke the tool using the following command: dotnet-dump
Tool 'dotnet-dump' (version '6.0.351802') was successfully installed.
root@e881fccccede:/# export PATH="$PATH:/root/.dotnet/tools"
```
- 進去dump的資料夾就可以做分析了
下指令`dotnet-dump analyze coredump.1`
```root@e881fccccede:/# cd /tmp/coredump
root@e881fccccede:/tmp/coredump# dotnet-dump analyze coredump.1
Loading core dump: coredump.1 ...
Ready to process analysis commands. Type 'help' to list available commands or 'help [command]' to get detailed help on a command.
Type 'quit' or 'exit' to exit the session.
>

```


[微軟docker image](https://learn.microsoft.com/zh-tw/dotnet/architecture/microservices/net-core-net-framework-containers/official-net-docker-images)<br/>
[參考文件](https://iter01.com/580653.html)<br/>
[dotnet dump](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump#dotnet-dump-analyze)<br/>
[.NET Core dump 分析](http://jinyazhou.com/dot_net_core_dump_analyse.html)

