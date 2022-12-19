---
title: "Redis基本指令與安裝"
date: 2022-12-19T15:04:32+08:00
draft: false
tags: ["redis"]
type: post
author: "Steve Lin"
description: ""
---

### 特性
- 儲存結構
    - 字串
    - 散列類型Hash
    - 列表類型List
    - 集合類型Set
    - 有序集合類型Sorted List
-  數據持久化，一般都在記憶體，也可以存到硬碟
-  可以用來作快取、隊列系統
-  可以設置TTL(time to live)，到期後會自動消失
-  單執行緒
-  支援publish/subscribe
-  可以限定資料占用的最大記憶體空間，達到限制後可以自動淘汰不需要的key
### 安裝Redis
- 次版本號偶數的是穩定版，奇數是非穩定版
- 使用docker安裝
```
docker pull redis
docker run --name redis-lab -p 6379:6379 -d redis

//確認container
docker ps
//測試 進bash 或是用docker desktop進入
docker exec -it e819 bash
//進bash之後輸入
redis-cli

# redis-cli
127.0.0.1:6379> ping
PONG
有回應就代表正常
```
- [在windows安裝](https://marcus116.blogspot.com/2019/02/how-to-install-redis-in-windows-os.html)
### 啟動Redis
- 可執行文件(應該是工具的意思)
```
redis-server Redis服務器
redis-cli 客戶端命令列
redis-benchmark 性能測試工具
redis-check-aof AOF文件修復工具
redis-check-dump RDB文件檢查工具
```
- 直接啟動 
進去bash後執行redis-server
但docker已經有用6379了 所以這邊是給非docker使用的
然後再執行`redis-server --port 6379`
- 透過redis_init_script執行(在Production環境用)
### 停止Redis
`redis-cli SHUTDOWN`
收到指令會先斷掉連線，執行持久化工作後退出
### 使用redis-cli發送指令
- redis-cli -h ``[host name]`` -p `[port]`可以指定redis server
`redis-cli -h [your host] -p 6379`
### 回傳值類型
- status reply:回復狀態
`OK或PONG`
- error reply:有錯誤的時候回傳error訊息
- integer reply (1或0):用(integer)開頭
- bulk reply(字串)
- multi-bulk reply (多重字串)
### 配置
- redis-server可以設定port、是否開啟持久化，日誌級別，可以透過文件設定
`redis-server /path/to/redis.conf`
- redis跟目錄有redis.conf，是提供的配置文件模板
- 可以動態修改配置，但不是所有的都可以修改
`CONFIG SET loglevel warning`

### 基本指令
- `keys pattern` 找符合條件的key
    - 可以用正規表達式 ?、*、`[]`、\x
- `exists key`判斷是否存在 存在回傳1
- `del key`刪除key 回傳值是key的個數(不支援regular)
- `type key` 取得key的數據類型
### 字串類型
- 可以存字串、二進位數據、json、圖片等等，最大容量是512MB
- key建議用`對象類型:對象ID:對象屬性`來命名 EX user:1:friends
- 如果是複雜Json資料可以用MessagePack壓縮
- `set key value`給值
- `get key`取值
- `incr key`遞增數字 回傳遞增後的值 失敗會傳傳`(error) ERR value is not an integer or out of range`
- `decr key`遞減數字
- `incrby key increment`  一次增加指定數值
- `decrby key decrement` 一次減少指定數值
- `incrbyfloat key increment`增加指定浮點數
- `append key value` 向尾部附加值 回傳增加後總長度
- `strlen key`取得字串長度
- `mget key [key...]` 同時取得多個值
- `mset ket value[....]`同時設置多個鍵值
### 散列類型Hash
- 結構是`KEY:字段:字段值 (key:field:value)` 字段值只能是字串
- 對象類別和ID構成KEY，字段表示對象屬性，字段值儲存屬性值
- `hset key field value `給值
- `hget key field`取值
- `Hmset key field value [field value]`設定好幾個field
- `Hmget key field [field…]`一次取得好幾個key
- `Hgetall key` 取得所有field value
- Hset不區分插入(insert)和更新(update)，如果插入前field不存在會返回1，如果是更新操作會返回0
- `Hexists key field`判斷field是否存在
- `Hsetnx key field value`當field不存在時給值(原子操作)，失敗回傳0
- `Hincr key field increment`增加數字
- `Hdel key field [field…]`刪除field
- `Hkeys key` `Hvals key`只需要取得field或field value
- `Hvals key`取得field數量
### 列表類型List
- 是雙向鍊結串列，兩邊元素的時間複雜度是O(1)
- 列表兩端增加元素
`Lpush key value [value…]`
`Rpush key value [value…]`
回傳值表示增加元素後的長度
- 列表兩端移除元素
`Lpop key` `Rpop key`
- 取得元素個數(會直接讀取現成的值，而不會尋訪所有元素統計) O(1)
`Llen key`
- `lrange key start stop`取得列表的指定元素值，start跟stop是index，從0開始
    - 支援負index EX:-1是最右邊第一個元素
    - Lrange key 0 -1 可以取得所有元素
    - 如果start<stop會回傳空列表如果stop大於實際的索引範圍，會回傳最右邊元素
- `Lrem key count value` 刪除列表中前count個值是value的元素
    - count>0 從左邊開始刪除
    - count<0從右邊開始刪除
    - count=0 刪除所有值為value的元素
- `Lindex key index` 取得指定index的元素
- `Lset key index value`針對index給值
- `Ltrim key start end`只保留指定範圍元素，指定索引之外的元素會刪除，可以用來限制元素數量
- `Linsert key before|after pivot value`插入元素在特定值(pivot)前或後，從左到右找到值為pivot的元素，把value插進去
- `rpoplpush source destination`把元素從一個list轉到另一個list，從source右邊取出元素放到destination左邊

### 集合類型Set
> 元素是沒有順序且唯一

- `Sadd key member [member…]`
`Srem key member [member…]`增加/刪除元素
- `Smembers key`取得所有元素
- `Sismember key member`判斷元素是否存在，時間複雜度O(1)
- 集合運算
    -  `sdiff key [key…]`差集
    ```
    {1,2,3}-{2,3,4} ={1}
    {2,3,4}-{1,2,3}={4}
    ```
    - `Sinter key [key…]`交集
    `{1,2,3}&{2,3,4}={2,3}`
    - `Sunion key [key…]`聯集
- `Scard key`獲得集合元素個數
- `Sdiffstore destination key [key…]`取差集再返回結果
`Sinterstore destination key [key…]` 跟上面類似
`Sunionstore destination key [key…]`
進行集合運算並儲存結果
- `Srandmember key [count]`隨機取得集合中元素
    - Count >0 隨機取不重複元素
    - Count >元素個數 回傳全部元素
    - Count <0 隨機取|count|元素 但有可能會一樣
- `Spop key`隨機取出一個元素

### 有序集合類型Sorted List
> 每個元素都有一個分數，可以取得分數最高或最低的前N個元素
- 跟List相同部分
    - 2者都是有序
    - 2者都可以獲得某個範圍的元素
-  跟List差別部分
    - 列表是linkerd list，取得兩端元素速度快，但元素變多後取得中間元素會變慢
    - sosrted list是靠hash跟skip list實做，所以取得中間元素也很快
    - 列表不能單獨調整元素位置，但有序集合可以(改分數)
    - 有序集合更消耗記憶體
- `Zadd key score member [score member…]`增加元素,如果member已存在，會用score替換原有分數,Score也可以用double
- `Zscore key member`取得元素分數
- `Zrange key start stop [withscores]`
`Zrevrange key start stop [withscores]`
    - 取得某個範圍內的元素列表,跟lrange類似 要同時取得分數可以加上withscores
    - 元素按照分數從小到大排,Zrevrange就是從大到小,分數一樣會按照字典順序
- `Zrangebyscore key min max [withscores] [limit offset count]`取得指定分數範圍的元素
    - 分數前加上”(“ 表示不包含 EX:80 (100   score>=80 && score<100
    - 正無窮大+inf
    - 負無窮大-inf
    - limit offset count表示要跳過offset的元素取count個
- `Zrevrangebyscore key max min [withscores] [limit offset count]`分數從大到小排列 min跟max也跟上面是相反的
- `Zincryby key increment member`增加某個元素分數
- `Zcard key`取得元素數量
- `Zcount key min max`取得指定分數範圍的元素個數
- `Zrem key member [member…]`刪除一個或多個元素
- `Zremrangebyrank key start stop`按排名範圍刪除元素(用index)
- `Zremrangebyscore key start stop`按分數範圍刪除元素
- `Zrank key member` `Zrevrank key member`取得元素排名(從0開始)
- `Zinterstore destination numkeys key [key…] [weights weight [weight…]] [aggregate sum|min|max]
`計算多個有序集合的交集，並把結果存在destination
```
zadd set1 1 a 2 b
zadd set2 10 a 20 b
 
zinterstore result 2 set1 set2
zrange result  0 -1 withscores
zrange result  0 -1 withscores

 zinterstore result 2 set1 set2 aggregate min
 srange result 0 -1
```

