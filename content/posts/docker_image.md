---
title: "Docker image"
date: 2022-10-24T15:35:38+08:00
draft: true
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---
# Docker image

### 指令

- 把image拉到本地端
`sudo docker pull`
- 查看image的詳細資訊
`sudo docker inspect [image name|image id]`
- 刪除docker image
`sudo docker rmi [image name] ` -f可以強制刪除
- 匯出image變成檔案
```
sudo docker save -o ubuntu.tar ubuntu:latest
sudo docker save ubuntu:latest > ubuntu.tar
```
- 匯入檔案變成image
```
sudo docker load -i ubuntu.tar
sudo docker load < ubuntu.tar
```
- 建立容器
`sudo docker run debian:jessie`
- 執行容器
`sudo docker run debian:jessie`
    - -i或-interative 互動模式
    - -t或-tty 開啟偽終端機
    - -d後臺執行 不能跟-i一起用
- 查看處理序狀態 比較像linux的ps用法
`docker top [container]`
- 查看容器資訊
    - `docker inspect [container]`
    - 可以用-f 過濾資訊
- 容器日誌
```docker logs [name]
docker run -d --name log_demo ubuntu /bin/bash -c 'for((i=0;1;i++)); do echo "time $i"; sleep 1;done;'
docker logs log_demo
只輸出最後幾行
docker logs  --tail 5 log_demo
限定時間
docker logs --since 2021-11-21T05:56:38 log_demo
```
- 附加到容器，就是切換成前台運行模式(依附在主處理序)，但是退出也會讓容器一起停止
`docker attach [name]`
- 容器中執行命令
`docker exec [name] `
- 提交容器更改
`docker commit [hash]`可以產生新的image
- 匯出容器
`docker export -o [name].tar [hash]`
- 匯入容器，但是匯入變成image
`docker import [name].tar`