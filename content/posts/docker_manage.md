---
title: "docker管理工具"
date: 2022-10-24T15:42:34+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---

# docker管理工具
## docker compose
- 組態預設放在docker-compose.yml檔案，也可以自訂檔名
- windows或MacOS已經內建安裝，用`docker-compose version`查看
- 其他OS要安裝需要去github抓檔案下來裝
- Docker compose是用yaml格式，可以上網找怎麼寫
### 範例
可以看kevin寫的[docker-compose.yaml](https://hackmd.io/@gs9TPhYbSPCyczQit5ucew/ryy_vyyJu)

### 文件規格
- services:代表所有組成專案的服務容器與配置
- image或build:代表容器用的基礎映像，build是用dockerfile產生的，值是dockerfile的目錄
- volume:跟-v用法一樣，但是可以指定相對於檔案的目錄
- environment:指定環境變數
- ports:指定連接port，最好用雙引號
- links:容器間的連接
### 指令
- `docker-compose build` 建構專案映像
	- --pull:總是拉最新的image
	- --firce-rm:刪除基於原有映像的容器
	- --no-cache:不使用快取
-  `docker-compose create` 建立專案容器
	- --force-create:重新建立已存在於docker的容器
	- --build:建構容器所需的映像
- `docker-compose start` 啟動所有容器
- `docker-compose start nginx php mysql` 指定要啟動的服務
- `docker-compose up` `docker-compose down`組合執行
- `docker-compose stop`停止專案
- `docker-compose rm`刪除專案
- `docker-compose logs <service>`取得服務日誌