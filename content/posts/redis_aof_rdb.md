---
title: "Redis管理與持久化"
date: 2022-12-19T15:36:54+08:00
draft: false
tags: ["redis"]
type: post
author: "Steve Lin"
description: ""
---

## 持久化方式
### RDB持久化
- 可以用save設定怎樣啟動snapshot
`save 900 1`900秒內有一個key被更改就進行snapshot
- snapshot文件儲存在目錄的dump.rdb文件裡，可以配置dir跟dbfilename參數指定snapshot儲存路徑跟文件名稱，redis啟動後會重新載入dump.rdb的資料到記憶體中恢復資料
- [過程](https://www.gushiciku.cn/pl/pmbW/zh-tw)
- 可以手動發送`save`或`bgsave`讓redis進行snapshot，save是透過main thread進行會*阻塞其他操作*，bgsave是fork子執行緒進行操作(也有可能會阻塞)
- 跟AOF比恢復資料比較快
- 用子執行緒操作，對redis server效能影響比較小
- 如果server當機，RDB會造成某個時間內資料遺失
### AOF持久化
> append only file
- 預設沒開啟，可以透過`appendonly yes`開啟
- 執行每個更改資料的指令都會寫到AOF文件
- AO文件跟RDB的位置一樣，預設是appendonly.aof，可以用`appendfilename appendonly.aof`修改
- 如果AOF記載太多冗餘命令，可以執行AOF重寫
    - `auto-aof-rewrite-percentage 100`:AOF文件超過上次重寫的百分比會再次重寫，如果沒重寫過會以啟動時的AOF文件大小當依據
    - `auto-aof-rewrite-min-size 64mb`: 限制允許重寫的最小AO文件大小
    - `bgrewriteaof`可以手動執行重寫
    - `no-appendfsync-on-rewrite no`預設不重寫aof檔案
- 每次啟動會把AOF文件的命令載入記憶體，速度比RDB慢
- 每次指令寫到AOF文件的時候不會馬上寫到硬碟，而是先寫到硬碟快取，每30秒執行一次同步操作，但如果同步失敗就會損失資料
    -  `appendfsync always|everysec(預設)|no` 
        -  always:客戶端的每一個寫操作都儲存到aof檔案當，這種策略很安全，但是每個寫請注都有IO操作，所以也很慢
        -  everysec:預設寫入策略，每秒寫入一次aof檔案，因此，最多可能會丟失1s的資料
        -  no:redis server不負責寫入aof，而是交由作業系統來處理什麼時候寫入aof檔案。更快，但也最不安全，不推薦使用

#### 如果只是把redis當作快取用，可以不考慮持久化

### redis複製replication
- 一台的資料更新後可以同步到其他server
- 分成master-slave，主數據庫可以讀寫操作，寫操作的時候會把資料同步到slave，從數據庫一般只負責讀
- slave只需要在配置文件加入`slaveof 主數據庫IP port`，master不用任何設定
- 可以改從數據庫的配置文件，`slave-read-only`改成no，讓從數據庫可以寫，但是更新資料不會同步，而且主數據庫更新會覆蓋掉從數據庫的資料
```
主數據庫
redis-server

從機監聽6379
redis-server --port 6378 --slaveof 127.0.0.1 6379

也可以運行時使用
slaveof 127.0.0.1 6379
```
- 流程
    - slave向master發送SYNC命令
    - master接收到命令後，開始保存snapshot(RDB持久化)，把保存期間接收到的命令保存起來
    - snapshot完成後會把文件跟所有命令發送給slave
    - slave接收到後會載入文件跟收到的命令 
- slave在同步時還可以接收其他client的命令，可以設定`slave-serve-stale-data`為no，讓同步完成前的命令都回傳錯誤
- slave也可以有自己的slave(多個)

### 安全機制
- redis server要限制不能被所有人讀取，要設置防火牆
- redis可以設定密碼，client每次連到redis都要發送，設定`requirepass`
- 可以重新命名指令，就可以保證只有自己可以用,`rename-command`
### 管理工具
- 耗時命令日誌:執行的指令超過時間限制會記錄，可以設定配置文件的`slowlog-log-slower-than`，單位是微秒(1000000微秒=1秒)，預設值是10000，日誌是記錄在記憶體，可以設定`slowlog-max-len`來限制紀錄筆數
    - 組成:log唯一ID,命令執行的UNIX時間,命令耗時時間,命令跟參數
```
slowlog get
```
- 命令監控:`monitor`,每個指令都會印出來，但很影響效能，所以只建議debug使用
```
127.0.0.1:6379> monitor
OK
1626156001.628433 [0 127.0.0.1:46324] "set" "foo" "bar"
```
- Rdbtools:可以把snapshot文件解析成json檔案