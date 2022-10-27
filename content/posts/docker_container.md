---
title: "docker container操作"
date: 2022-10-24T15:45:01+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---
# docker container操作
## 進container改檔案

### 方法一
1.docker exec -it <container id> bash
2.apt-get update
3.apt-get install vim
4.vim <檔名>
5.用insert模式修改，完成後Esc + :wq
    
### 方法二
1.docker exec -it <container id> bash
看配置跟檔案位置
2.docker cp <container id>:<file path+file name> <host file path>
3.改完檔案內容之後複製回去
docker cp  <host file path +file name> <container id>:<file path>

## 刪除container
docker container rm [hash]
## 刪除image
docker image rm [hash] 

https://blog.clarence.tw/2019/09/10/docker-removing-containers-images-volumes-and-networks/