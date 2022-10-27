---
title: "Docker介紹"
date: 2022-10-24T15:34:48+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---
# Docker

### 底層實作
> 使用 Linux Kernel的 Namespace: 和 Cgroup 組合而成
- Namespace:作環境隔離使用的，像是把Process ID、User ID、Network…等等的環境執行狀態隔離開來
![](https://ithelp.ithome.com.tw/upload/images/20171205/20103456ApLF6SQff3.png)
- Cgroup:用來做系統資源的管理，如：CPU、Memory、I/O…等等，資源做有效的隔離和做使用上的限制

### 系統架構
> client-server的架構，client部分稱為docker client，server的部分稱為docker daemon
- Docker Daemon：用來執行管理Docker image、啟動 container、停止 container的service，有提供 Restful API 給使用者做操作或是顯示一些 Docker container 的狀態訊息
- Docker Client:使用 Restful API 連到 Docker daemon，並且提供 command line 的方式讓使用者可以操作 docker

### 常用名詞
- docker image:啟動docker container要使用的擋案，是一個唯讀映像檔，可以像積木一樣一層一層堆起來
    - 可以從Docker Hubpull下來
    - 從另外一台電腦上的 Docker image export 出來，然後import到自已的電腦
    - 寫dockerfile
![](https://ithelp.ithome.com.tw/upload/images/20171205/20103456jl9BuRvKSl.png)
- Docker Container:透過 Docker image 執行起來的 Process，同一個 Docker image 可以啟動多個 Docker Container，container之間的環境是隔離開來的
- Docker Hub:存放 Docker image的倉庫，內部環境架的私有docker hub叫Docker Registry

[小練習](https://ithelp.ithome.com.tw/articles/10190921)

### 指令介紹
- docker search:搜尋有哪些image
    -  is-official=true:找官方的image
- docker pull [imagename]:把image下載到本地端
- docker images:檢查現有的image
- docker run -it [imagename]:執行container
    - /bin/bash:可以進入terminal 輸入exit可以離開(會停止container)，`ctrl + p`+`ctrl + q`不會把container關閉
- docker ps :查看現有的container
    - -a:包含停止的
- docker exec -it [containerid] /bin/bash進入terminal模式
### Dockerfile
> 不需要進container安裝程式和設定檔，只要`docker build`就可以建docker image，之後要用就只要`docker run`

### 如果要分享給別人使用image
- 直接把 Dockerfile 傳給其他人用，但是拿到 Dockerfile 之後要花一點時間重新 Build Image
- 把 Docker Image 放在公開或私有的網路上給人 Pull 下來使用
- 在沒有網路環境的狀況下適合使用 Import/Export 檔案的方式
### 把docker image 推到Docker Hub
- 要先註冊docker hub帳號 [連結](https://hub.docker.com/)
- 把docker image push上去
    - `docker images`查看現有的images
    - 要推上去需要加上tag 
docker tag ${Image Name} DockerHub帳號/Image Name
EX: `docker tag ubuntu shadow061103/ubuntu`
- 輸入`docker login`登入docker hub
```
docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: shadow061103
Password:
Login Succeeded
```
- 輸入` docker push [OPTIONS] NAME[:TAG]`推上去
```
docker push shadow061103/ubuntu
```
- 登進去Docker Hub看應該就會看到
- 可以先`docker rmi image`刪除掉再用`docker pull`拉下來
