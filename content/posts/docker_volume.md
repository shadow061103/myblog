---
title: "docker的資料卷Data Volume"
date: 2022-10-24T15:40:10+08:00
draft: true
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---

# docker的資料卷Data Volume
> 保存檔案資料的模組
- 容器內的檔案環境是在一個臨時層上，容器停止或刪除時，檔案就不見了
- 資料卷是掛載在容器內檔案系統的檔案或目錄，可以永久保存
- 不依賴於容器，所以容器可以共用

### 指令
- 對資料卷指定名稱，不然就要用hash才能取得
```
 docker volume create --name html
```
- 掛載volume在容器
`docker create --name web -v /html -v /var/log/nginx nginx`
	- -v [name]:[dest] : name是資料卷名稱 dest是目錄
	- 也可以把server的目錄當作資料卷掛到容器上，用 -v [src]:[dest]
	- 後面加:ro可以變唯讀檔案
	- 範例是掛2個volume
- 檢查
```
docker volume inspect html
[
    {
        "CreatedAt": "2021-11-30T15:10:52Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/html/_data",
        "Name": "html",
        "Options": {},
        "Scope": "local"
    }
]
```

- 查看所有的
`docker volume list`
- 刪除資料卷(沒被任何容器使用時才會刪)
```
docker volume rm [hash]
或是隨容器刪除
docker rm -v web
```

### 資料卷容器
- 用來管理資料卷，是其他容器的橋樑
- 不用處在運行狀態也可以用
- 其他容器只是透過資料卷容器取得的資料卷資訊
```
docker create --name data -v /html ubuntu
docker inspect data
//掛載指定資料卷的所有資料卷
docker run -d --name web --volumes-from data nginx
```
- --volumes-from指定要掛哪一個
- 同一個資料卷容器內的檔案會共享
### 要匯出資料卷
- 先建立一個新的container
- 用--volumes-from指定資料捲容器
- -v指定本地端檔案
- 建立之後把包成tar檔
- 關閉container

[Docker筆記 - 讓資料遠離Container，使用 Volume、Bind Mount 與 Tmpfs Mount](https://medium.com/alberthg-docker-notes/docker%E7%AD%86%E8%A8%98-%E8%AE%93%E8%B3%87%E6%96%99%E9%81%A0%E9%9B%A2container-%E4%BD%BF%E7%94%A8-volume-bind-mount-%E8%88%87-tmpfs-mount-6908da341d11)