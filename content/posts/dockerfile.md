---
title: "Dockerfile"
date: 2022-10-24T15:42:59+08:00
draft: true
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---

# dockerfile介紹
### 介紹
- #開頭的是註解
- 指令分成指令名稱，然後是args
`instruction arguments`
- `# key=value`的是設定參數
- `docker build ~/path`可以建image，後面是指定檔案目錄(不是file檔路徑)，目錄底下要有Dockerfile
	- 可以用-t指定名稱跟tag(跟建container一樣)
	- 如果名稱不是Dockerfile可以用-f指定檔案名稱
	- `docker build -t mytag .` 後面有.表示用這個資料夾的檔案
	
### 基礎指令
- FROM
	- 指定建置的image是基於哪個image而來
	- 格式
	 ```
	 FROM <image>
	 FROM <image>:<tag>
	 FROM <image>@<digest>
	 tag跟digest都是optional的，沒填就用latest
     FROM <image> as <name>這個name可以自訂 之後用COPY --from指定
	``` 
- MAINTAINER
提供工作者資訊
`MAINTAINER <name>`
### 控制指令
- RUN
	- RUN command para1 para2是用shell程式執行指令，可以支援換列,
	實際上是用/bin/sh執行
	- 如果指令一樣但結果不同，docker會用快取的結果，所以可以指定`--no-cache`解決
	- 建構映像的過程中執行，並把結果提交到新的映像層中
```
RUN command para1 para2... //比較好
RUN ["executable","para1","para2",...]
```
	
- WORKDIR
用來切換工作目錄，可以是相對或絕對路徑
允許使用環境變數
```
WORKDIR /usr
WORKDIR local
用相對路徑會參考目前工作目錄去切換
```
- ONBUILD
可以攜帶另一個指令，但不會在建構目前映像的時候執行
而是在建構其他映像，並透過FROM把目前映像當作基礎映像的時候執行
可以用docker inspect查看有哪些指令
### 引入指令
- ADD
可以把source code或設定檔之類的東西匯入
可以用go的萬用字元比對規格指定多個檔案`ADD hom* /dir/`
src只能用相對路徑，而且是相對於dockerfile的路徑，不能往上層找(就是只能找dockerfile下的檔案)
dest可以是相對或絕對路徑，
src可以放網址下載外部網路的內容
```
ADD <src>... <dest>
ADD ["<src>",..."<dest>"]
```
- COPY
跟ADD一樣，差在不能識別網路位址，跟解壓縮檔案
```
COPY <src>... <dest>
COPY ["<src>",..."<dest>"]
```
### 執行指令
- CMD
會指定容器中的主體程式，所以應該只會有一個CMD指令
目的是設定映像的預設入口程式
```
CMD command para1 para2...
CMD ["executable","para1","para2",...] //比較好 執行的是程式本身
CMD ["para1","para2",...] //把參數傳給evtrypoint列出的程式
```
建容器時可以覆蓋掉CMD指令指定的程式
`docker run -it nginx /bin/bash`
- ENTRYPOINT
有用ENTRYPOINT的時候，CMD指令或docker run的指令會當作參數，串接到ENTRYPOINT原有的命令後(CMD避免使用shell執行)
反正不懂就看書
```
ENTRYPOINT command para1 para2...
ENTRYPOINT ["executable","para1","para2",...]
```
### 配置指令
- EXPOSE
跟docker run用的-p或-P不一樣，是列出此映像的容器需要開放的port，以便從容器外部存取
從宿主機外部存取是用-p或-P
從其他容器存取是用--link連接到此容器
```
EXPOSE <port> [<port>...]
```
- ENV
設定環境變數用
docker inspect可以查看
docker run --env 可以新增和替換環境變數
ENV設定的可以繼承，類似全域變數的效果
也可以在RUN的時候指定`RUN <key>=<value>`
```
ENV <key> <value>
ENV <key>=<value>...

```
- LABEL
中繼資料的標記 可以用\換列
有繼承關係，可以全域使用，後面會蓋掉前面
```
LABEL <key>=<value> <key>=<value> <key>=<value>
```
- USER
設定使用者
`USER nginx`
- ARG
ENV會影響映像編譯，也會體現在容器運行(container裡面)
ARG只適用於映像的建構過程，效果不會作用於基於此映像的容器
docker run可以用`--build-arg <key>=<value>`設定
ENV設定的會永遠覆蓋ARG 使用時一樣用$name
```
ARG <name> //只是宣告變數 沒有給定值
ARG <name>=<default>
```
- SHELL
用來指定SHELL程式
預設是/bin/sh
`SHELL ["executable","para"]`
### 指令解析
換列分隔符號\適用於Linux或Mac OS系統
windows系統可能會出錯
可以用`#escape='`重新定義分隔符號，這樣dockerfile就不會出錯
### 忽略檔案
建一個.dockerignore檔案放要忽略的副檔名

### .NET使用dockerfile啟動專案
```
在專案跟目錄下語法
docker build -t test -f src/Presentation/PXPlus.CreditCard.Scheduler/Dockerfile . --no-cache
docker run --restart=always -d -p 9079:9079 -e "ASPNETCORE_ENVIRONMENT=Development" --name test-creditcard-scheduler test
```