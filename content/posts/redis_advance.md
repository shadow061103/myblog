---
title: "Redis進階用法"
date: 2022-12-19T15:24:31+08:00
draft: false
tags: ["redis"]
type: post
author: "Steve Lin"
description: ""
---
### Transaction
- 全執行或全部執行(ACID)
- 先用`multi`告訴redis要執行事務的指令,會先進到queue等待
- `exec`才會全部執行,回傳值是依照指令順序
- 沒有rollback機制
```
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> sadd user:1:following 2
QUEUED
127.0.0.1:6379(TX)> sadd user:2:followers 1
QUEUED
127.0.0.1:6379(TX)> exec
1) (integer) 1
2) (integer) 1
127.0.0.1:6379>
```
- 有語法錯誤就不會執行
```
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set key value
QUEUED
127.0.0.1:6379(TX)> set key
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379(TX)> errorcommand key
(error) ERR unknown command `errorcommand`, with args beginning with: `key`,
127.0.0.1:6379(TX)> exec
(error) EXECABORT Transaction discarded because of previous errors.
```
- 如果是執行時錯誤,其他指令還是會被執行
```
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set key 1
QUEUED
127.0.0.1:6379(TX)> sadd key 2
QUEUED
127.0.0.1:6379(TX)> set key 3
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) OK
127.0.0.1:6379> get key
"3"
```
### Watch
- 可以監控一個或多個key，有一個被修改或刪除，之後的交易就不會執行
- 監控一直持續到exec執行後，所以watch可以修改watch監控的key-value
- 在交易中想取消監控可以用`unwatch`

### Timeout時間
- `expire key seconds`設定生存時間
- `ttl key`看剩餘時間(秒) 回傳-1表示沒有設定時間
- `persist key`取消時間設定(永久)
- 用`expire`可以重新設定時間或用`set`取消生存時間
- `pexpire key` 單位是毫秒 `pttl`可以看剩餘時間
- 如果用watch監控有生存時間的key，自動刪除不會被watch認為改變
- 記憶體有限的情況下，要設定快取能用的最大值，可以透過修改`maxmemory-policy`來指定策略

### 排序
- `sort`可以排序List、Set、Sorted List的類型
- 對有序集合排序會忽略分數，只對值排序
- 可以針對字典順序做排序
```
127.0.0.1:6379> lpush list a c e d B C A
(integer) 7
127.0.0.1:6379> sort list
(error) ERR One or more scores can't be converted into double
127.0.0.1:6379> sort list alpha
1) "A"
2) "B"
3) "C"
4) "a"
5) "c"
6) "d"
7) "e"
```
- `sort key desc`可以降冪排序
- 可以搭配`limit offset count`使用
- 時間複雜度是O(n+mlogm) (n是排序的列表元素個數，m是要回傳的元素個數)
- 要調整排序性能要盡量減少排序的key中元素數量，或是用limit只取得需要的數據
#### By參考鍵
- 可以是字串類型或Set的字段，sort可以依照by的欄位排序
```
127.0.0.1:6379> lpush sortbylist 2 1 3
(integer) 3
127.0.0.1:6379> set itemscore:1 50
OK
127.0.0.1:6379> set itemscore:2 100
OK
127.0.0.1:6379> set itemscore:3 -10
OK
127.0.0.1:6379> sort sortbylist by itemscore:* desc
1) "2"
2) "1"
3) "3"
```
- by的參考鍵名要有`*`不然就跟一般取range的方式一樣不會排序(*只能在key部分才有用)
- 如果參考鍵的值都一樣，那會比較元素本身的值
- 參考鍵不存在的話，值會當成0，
#### Get參數
- 在sort後面加Get參數可以取得指定的鍵值，可以有多個get
- `get #`會返回元素本身的值
#### Store參數
- 可以保存排序後的結果(List)
### 任務佇列
- 有producer跟consumer，producer會發送任務事件，consumer會去接收並執行
- `brpop key [key...] timeout` 
`blpopkey [key...] timeout` 可以監聽多個key，後面時間如果設成0表示會阻塞操作，直到取到值才會往下做
```
redis-A
127.0.0.1:6379> blpop queue:1 queue:2 queue:3 0
1) "queue:2"
2) "task"
(13.59s)
redis-B
127.0.0.1:6379> lpush queue:2 task
(integer) 1
```
### 發佈訂閱模式
- 發佈者可以向指定的頻道發送訊息，有訂閱的都會收到
- 訂閱者可以訂閱多個頻道channel
- `publish channel message`發佈消息，回傳值是收到訂閱的數量
- `subscribe channel [channel...]`訂閱頻道，執行後進入訂閱狀態，只能使用`subscribe/unsubscribe/psubscribe/punsubscribe`這幾個命令
- 會收到3種類型的回覆
    -  subscribe:訂閱成功的訊息+訂閱成功的頻道名稱+目前client端訂閱的頻道數量
    -  message:接收到的訊息+產生訊息的頻道名稱+訊息內容
    -  unsubscribe:成功取消訂閱某頻道+對應頻道名稱+目前訂閱數
```
subscribe
127.0.0.1:6379> subscribe channel.1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel.1"
3) (integer) 1
1) "message"
2) "channel.1"
3) "hi"
1) "message"
2) "channel.1"
3) "fuck"
127.0.0.1:6379> unsubscribe channel.1
1) "unsubscribe"
2) "channel.1"
3) (integer) 0

publisher
127.0.0.1:6379> publish channel.1 hi
(integer) 0
127.0.0.1:6379> publish channel.1 hi
(integer) 1
127.0.0.1:6379> publish channel.1 fuck
``` 
- `psubscribe`可以訂閱指定名稱的頻道，支援glob匹配
    - 可以重複訂閱同一個頻道，接收到的message也就會有2個 
```
127.0.0.1:6379> psubscribe channel.?*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "channel.?*"
3) (integer) 1
```
- `punsubscribe [pattern...]`退訂指定的規則，但只限於psubscribe訂閱的

### Pipeline
- 可以一次發送多個命令給redis，執行完後一次把結果回傳，但每個命令彼此要獨立
- 有效減少網路來回延遲的時間
- [圖片說明](https://jed1978.github.io/2018/05/11/Redis-Programming-CSharp-Basic-1.html)
### 節省記憶體空間
- 因為redis是存在記憶體，而記憶體比起硬碟相對貴
- 精簡key name跟value值
- 內部code優化，redis會自動調整，可以用`object excoding key`看

### 腳本
- 用Lua寫的,redis用5.1版
- 減少網路開銷
- 原子操作，不用使用transaction
- 可以重複使用
- 用法
    - 寫完腳本後存成.lua檔案，然後下指令
    - `redis-cli --eval /path/to/ratelinit,lua rate.limiting:127.0.0.1, 10 3`
    - `rate.limiting:127.0.0.1`是要操作的key，可以用`keys[1]`取得
    - 後面的10和3是參數，可以用`ARGV[1]`和`ARGV[2]`取得
- 常用型別
    - 空nil:沒給值的變數或table的字段
    - boolean
    - number:整數或浮點數
    - string:可以用'或"
    - 表table:可以當字典或數據結構
    - function:可以儲存在變數中,做為函數的參數或回傳值
- 變數:只能用區域變數，因為全域變數可能影響腳本之間，宣告的時候用local，只能用英文字母、數字、下底線，區分大小寫，不能用保留字以及數字開頭 
```
local c --沒給值就是nil
local d = 1
local e, f --可以宣告多個
```
- 註解:單行用`--`，多行用`--[[開始 ]]結束`
- 給值，(列表類型index從1開始)
```
local a, b = 1, 2
local c, d = 1, 2, 3 --3被捨棄
```
- 運算子:+,-,*,/,%,-(負),^(冪)，
`'1'+1=2 ,'10'*2=20 跟js一樣會自動轉換`
- 比較運算:==,~=(不相等),>,<,>=,<= (比較運算不會自動轉型)
- 邏輯運算:not,and,or，只要不是nil或false就會當作true，包含0跟true，所以用exist回傳的0或1結果都是true，要改用==1判斷
```
1 and 5 --5
1 or 5 --1
not 0 --false
'' or 1 -- ''
```
- 連接操作:用`..`連接字串
```
'hello' .. 'world!'
'the price is' .. 25 --會把數字轉成字串
```
- 取長度操作:用# `print(#'hello') --5`
- 不用縮排也不一定要用;結尾
- 迴圈:while,repeat,for，有用到再找用法就好，可以用`ipairs`或`pairs`做iterator
    -  `ipairs`:只會從1開始巡覽到最後一個不為nil的整數索引
    -  `pairs`:會巡覽所有值不為nil的索引
```
repeat
 ...
until

while condition do
    ...
end

for var=start ,end,step do
    ...
end
```
- table類型:就是js的物件類別，可以用["name"]的方式取值，index是從1開始
```
a = {}
a['field']='value'//set value
people={
name='Bob',
age=29
}

```
- 函數
```
function ()
    ...
end

local square= function(num)
    return num * num
end
--省略
local function square(num)
return num * num
end
```
- Lua標準函式庫:Base,String,Table,Math,Debug
- 其他函式庫:cjson跟cmsgpack可以用來序列化成json或msgpack
- 用redis命令要下redis.call
`redis.call('set','foo','bar')`
### 腳本指令
 - EVAL:`eval [key...] [arg...]` 可以在腳本中用KEYS跟ARGV取得
  ` EVAL "return redis.call('set',KEYS[1],ARGV[1])" 1 foo bar`
- EVALSHA:會把腳本內容替換成SHA1
- `script load` :把腳本加到快取
- `script exists`:判斷腳本是否有快取
- `script flush`:清空腳本快取
- `script kill`終止當前腳本的執行
- 原子性和執行時間:執行腳本期間不會執行其他指令，所有指令都要等腳本完成才能執行，`lua-time-limit`可以設定腳本執行時間(預設5秒)，執行腳本期間下其他指令會得到busy的錯誤，這時候只能下script kill或shutdown nosave，但如果腳本有對資料做修改是不能kill的，只能用shutdown的語法強制終止操作(不會持久話操作)
```
BUSY Redis is busy running a script. You can only call SCRIPT KILL or SHUTDOWN NOSAVE.

被終止的腳本會回傳錯誤
(error) ERR Error running script (call to f_694a5fe1ddb97a4c6a1bf299d9537c7d3d0f84e7): @user_script:1: Script killed by user with SCRIPT KILL...
```
- 腳本不應該放需要耗時的運算，因為redis是單執行緒