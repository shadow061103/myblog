---
title: "Sql Guideline"
date: 2022-11-25T15:51:13+08:00
draft: false
tags: ["DB"]
type: post
author: "Steve Lin"
description: ""
---
# SQL guideline
詳細可參考 https://www.sqlstyle.guide/
由於每個環境應用程度及方式不同，以下列出最少需要的部分

2022/03/31 更新  將Table/索引相關的部分移至 資料表與索引設計
2022/04/06 新增  說明Bitwise AND用法
1.系統保留字應盡量使用全大寫字元，使用到系統函式也需使用全大寫
詳細內容可參考 https://docs.microsoft.com/en-us/sql/t-sql/language-elements/reserved-keywords-transact-sql?view=sql-server-ver15
基本最少需要將SQL的保留字使用全大寫的方式

Mysql保留字
```
ADD
EXTERNAL
PROCEDURE
ALL
FETCH
PUBLIC
ALTER
FILE
RAISERROR
AND
FILLFACTOR
READ
ANY
FOR
READTEXT
AS
FOREIGN
RECONFIGURE
ASC
FREETEXT
REFERENCES
AUTHORIZATION
FREETEXTTABLE
REPLICATION
BACKUP
FROM
RESTORE
BEGIN
FULL
RESTRICT
BETWEEN
FUNCTION
RETURN
BREAK
GOTO
REVERT
BROWSE
GRANT
REVOKE
BULK
GROUP
RIGHT
BY
HAVING
ROLLBACK
CASCADE
HOLDLOCK
ROWCOUNT
CASE
IDENTITY
ROWGUIDCOL
CHECK
IDENTITY_INSERT
RULE
CHECKPOINT
IDENTITYCOL
SAVE
CLOSE
IF
SCHEMA
CLUSTERED
IN
SECURITYAUDIT
COALESCE
INDEX
SELECT
COLLATE
INNER
SEMANTICKEYPHRASETABLE
COLUMN
INSERT
SEMANTICSIMILARITYDETAILSTABLE
COMMIT
INTERSECT
SEMANTICSIMILARITYTABLE
COMPUTE
INTO
SESSION_USER
CONSTRAINT
IS
SET
CONTAINS
JOIN
SETUSER
CONTAINSTABLE
KEY
SHUTDOWN
CONTINUE
KILL
SOME
CONVERT
LEFT
STATISTICS
CREATE
LIKE
SYSTEM_USER
CROSS
LINENO
TABLE
CURRENT
LOAD
TABLESAMPLE
CURRENT_DATE
MERGE
TEXTSIZE
CURRENT_TIME
NATIONAL
THEN
CURRENT_TIMESTAMP
NOCHECK
TO
CURRENT_USER
NONCLUSTERED
TOP
CURSOR
NOT
TRAN
DATABASE
NULL
TRANSACTION
DBCC
NULLIF
TRIGGER
DEALLOCATE
OF
TRUNCATE
DECLARE
OFF
TRY_CONVERT
DEFAULT
OFFSETS
TSEQUAL
DELETE
ON
UNION
DENY
OPEN
UNIQUE
DESC
OPENDATASOURCE
UNPIVOT
DISK
OPENQUERY
UPDATE
DISTINCT
OPENROWSET
UPDATETEXT
DISTRIBUTED
OPENXML
USE
DOUBLE
OPTION
USER
DROP
OR
VALUES
DUMP
ORDER
VARYING
ELSE
OUTER
VIEW
END
OVER
WAITFOR
ERRLVL
PERCENT
WHEN
ESCAPE
PIVOT
WHERE
EXCEPT
PLAN
WHILE
EXEC
PRECISION
WITH
EXECUTE
PRIMARY
WITHIN GROUP
EXISTS
PRINT
WRITETEXT
EXIT
PROC
```
2.
使用空格和縮排來增強可讀性
EX:
SQL語法範例
~~~sql
-- 系統保留字向左對齊
SELECT  A.[receiver_ep_account_id],
        B.[ep_account],
        COUNT(A.order_id) AS [Orders]
FROM [dbo].[receive_order] AS A WITH (NOLOCK)
    INNER JOIN [dbo].[ep_account] AS B WITH (NOLOCK)
        ON A.[receiver_ep_account_id] = B.[account_id]
WHERE
    [receiver_ep_account_id] < 100
    AND [collection_balance] = 9000000.00
GROUP BY [receiver_ep_account_id], B.[ep_account]
ORDER BY [receiver_ep_account_id] ASC
;
~~~
範例
~~~sql
-- 系統保留字向右對齊
SELECT  A.[receiver_ep_account_id],
        B.[ep_account],
        COUNT(A.order_id) AS [Orders]
  FROM [dbo].[receive_order] AS A WITH (NOLOCK)
    INNER JOIN [dbo].[ep_account] AS B WITH (NOLOCK)
        ON A.[receiver_ep_account_id] = B.[account_id]
 WHERE
    [receiver_ep_account_id] < 100
    AND [collection_balance] = 9000000.00
 GROUP BY [receiver_ep_account_id], B.[ep_account]
 ORDER BY [receiver_ep_account_id] ASC
;
~~~
3.避免使用隱含轉換，容易造成CPU飆升及Column進行Datatype轉換比對，應明確使用相對的DataType
Column Datatype是字串型態則一定要加兩個單引號，數字型態一定不要加單引號
另外一個缺點是等號左邊會換成與等號右邊相同Datatype之後才開始比對
若左邊是欄位則整張表該欄位會先換成右邊的相同Datatype
在資料表量級大的時候會造成非常大量的記憶體使用及延遲
EX:

~~~sql
--[dbo].[ep_account]此表member_id為bigint(int64)
--此為正確查詢方式
SELECT  [id]
        ,[member_id]
        ,[account_id]
        ,[identity_no]
FROM [dbo].[account] WITH (NOLOCK)
WHERE
    [member_id] = 2
;
 
--以下merber_id查詢條件指定為字串，為造成隱含轉換，應避免使用
SELECT  [id]
        ,[member_id]
        ,[account_id]
        ,[identity_no]
FROM [dbo].[account] WITH (NOLOCK)
WHERE
    [member_id] = '2'
;
~~~
4.指定欲查詢的筆數，在對外服務的資料庫上若無法明確知道會有多少筆數據會被SQL篩選出來則使用指定筆數
EX:

~~~sql
-- TOP (expression) 一定不要省略 ()
-- 明確指定 TOP (n) 避免後續應用上一些效能及問題
SELECT  TOP (100) [id]
        ,[member_id]
        ,[account_id]
        ,[identity_no]
FROM [dbo].[account] WITH (NOLOCK)
;
~~~

MySQL取筆數
~~~sql
-- LIMIT n 直接取筆數
-- LIMIT start_row n 從第幾筆之後取筆數
SELECT  `id`
        ,`name`
        ,`term_type`
        ,`is_active`
FROM `terms`
LIMIT 10
;
~~~
5.避免兩大表Join，若無法避免則需先依照過濾條件縮小第一張表大小
理論是大表Join小表再進行過濾條件以一次性大量IO執行，但實際上應該反過來由小表Join大表，先將第一張表依照過濾條件限縮到最小範圍才Join
必須Join多張表時，請依序由最左(或最上)的表開始限縮，以最小資料量方式依序Join其他資料表

EX:
大表JOIN小表
~~~sql
--以下為假想情境，假設MSSQL此兩張表為有關連性必須使用JOIN方式查詢
--假設pxplus_paymentrecord是一張很大的資料表，Account也是一張很大的資料表
--先從可以限縮的表開始進行，變成是小表JOIN大表的方式
SELECT  TOP (1000)
        A.[Id]
        ,A.[MemberId]
        ,A.[StoreMemberId]
        ,A.[TransRecordType]
        ,A.[TransRecordSubType]
        ,A.[TransStatus]
        ,A.[OrderNo]
FROM (
        SELECT  [Id]
                ,[MemberId]
                ,[StoreMemberId]
                ,[TransRecordType]
                ,[TransRecordSubType]
                ,[TransStatus]
                ,[OrderNo]
        FROM [dbo].[trans_record] WITH (NOLOCK)
        WHERE
            [MemberId] = 4
    ) AS A
    INNER JOIN [dbo].[order] AS B WITH (NOLOCK)
        ON A.[OrderNo] = B.[order_no]
~~~
6.
查詢資料時需要明確指定Nolock Hint
EX:
MSSQL Hint
~~~sql
SELECT  TOP (100) [id]
        ,[member_id]
        ,[account_id]
        ,[identity_no]
FROM [dbo].[account] WITH (NOLOCK)
;
~~~
7.
SQL語法撰寫需盡量符合SARG格式
符合：=、<、>、>=、<=、BETWEEN、LIKE (若%在最左邊或中間則不符合且會導致DB engine不使用索引，最右邊符合)
不符合：NOT、!=、<>、!>、!<、NOT EXISTS、NOT IN、NOT LIKE

8.
IN 與 EXISTS使用差異
|in | exists | 
| - | - |
|IN等同於OR運算符|要確定是否返回任何值時使用EXISTS|
|當subquery結果比主要查詢Table較小時，IN會比EXISTS快 | 當subquery結果比主要查詢Table較大時，EXISTS會比IN快|
|DB engine會比較所有IN中的值|當EXISTS條件中為true(任何一筆符合)，DB engine將會停止往下繼續查詢|
|僅比對單個欄位時使用 |檢查多個欄位時使用|
|不能將內容與NULL值比較，將不返回任何結果|subquery中的所有內容與NULL值比較，將會返回差異結果|
|可以給出一組字串進行比較或使用subquery|必須給出subquery才能使用|
|主表與內表(subquery中的表)使用hash連結|對主表作loop，每次loop在進行內表關聯子查詢|

|not in | not exists | 
| - | - |
|主表與內表(subquery中的表)使用hash連結 |對主表作loop，每次loop在進行內表關聯子查詢 |
|不能將內容與NULL值比較，將不返回任何結果 |subquery中的所有內容與NULL值比較，將會返回差異結果 |
|主表與內表將全表掃瞄 | 能使用到相關索引|
9.不使用舊式Join改用ANSI 92及之後的標準Join方式，要明確指定Join方式
EX:
~~~sql
--舊式Join
SELECT  TOP (1000)
        A.[Id]
        ,A.[MemberId]
        ,A.[OrderNo]
FROM [dbo].[trans_record] AS A, [pxplus_account].[dbo].[order] AS B
WHERE
    A.[OrderNo] = B.[order_no]
    AND [MemberId] = 4
;
 
--標準JOIN方式
SELECT  TOP (1000)
        A.[Id]
        ,A.[MemberId]
        ,A.[OrderNo]
FROM [dbo].[trans_record] AS A WITH (NOLOCK)
    INNER JOIN [pxplus_account].[dbo].[order] AS B WITH (NOLOCK)
      ON A.[OrderNo] = B.[order_no]
WHERE
    [MemberId] = 4
;
~~~

10.匯總數量使用SUM取代COUNT
SUM使用記憶體，COUNT使用CPU
若只單純需要計算數量則盡量不使用COUNT
EX:
使用COUNT
~~~sql
SELECT  A.[MemberId]
        ,COUNT(A.[Id]) AS Cnt
FROM [dbo].[trans_record] AS A WITH (NOLOCK)
WHERE
    [MemberId] = 4
GROUP BY A.[MemberId]
;
~~~
使用SUM
~~~sql
SELECT  A.[MemberId]
        ,SUM(CASE WHEN A.[Id] > 0 THEN 1 ELSE 0 END) AS Cnt
FROM [dbo].[trans_record] AS A WITH (NOLOCK)
WHERE
    [MemberId] = 4
GROUP BY A.[MemberId]
;
~~~
11.欄位或資料表別稱，必須明確指定使用AS
EX:
別稱關鍵字
~~~sql
--未明確指定AS
SELECT  A.[MemberId]
        ,COUNT(A.[Id]) Cnt
FROM [dbo].[trans_record] A WITH (NOLOCK)
WHERE
    [MemberId] = 4
GROUP BY A.[MemberId]
;
 
--明確使用AS
SELECT  A.[MemberId]
        ,SUM(CASE WHEN A.[Id] > 0 THEN 1 ELSE 0 END) AS Cnt
FROM [dbo].[trans_record] AS A WITH (NOLOCK)
WHERE
    [MemberId] = 4
GROUP BY A.[MemberId]
;
~~~

12.
查詢需明確使用到索引搜尋
避免全表掃描甚至大量耗用CPU，在查詢條件上若無法使用索引或無法使用索引搜尋，則取已有的欄位進行並且增加相關假的查詢條件，
讓SQL解析有機會能走向所需的索引並且使用索引搜尋
請參考 https://docs.microsoft.com/zh-tw/sql/relational-databases/in-memory-oltp/indexes-for-memory-optimized-tables?view=sql-server-ver15
應用上經常有條件不同的部分，在依照需求定例的條件經常會使得DB engine不使用索引，下圖為微軟說明
![](https://imgur.com/UEr1iiN.jpg)
EX:
正常符合索引查詢
~~~sql
SELECT  [id]
        ,[order_no]
FROM [dbo].[order] WITH (NOLOCK)
WHERE
    [order_no] = 'PX202105200921460368'
    AND [order_type] = 1
;
 
--執行計畫，有效使用索引並且以最佳效率index seek(索引搜尋)
~~~
![](https://imgur.com/jeIBH8j.jpg)
然而情境轉換，可能只需要查詢 [order_type]而不需要[order_no]時，先來看看如果只查詢[order_type]

查詢索引中已包含的欄位卻無法使用索引搜尋
~~~sql
SELECT  [id]
        ,[order_no]
FROM [dbo].[order] WITH (NOLOCK)
WHERE
    [order_type] = 1
;
 
--執行計劃顯示，雖有使用索引但並非搜尋而是掃描
~~~
![](https://imgur.com/ALMIfxU.jpg)
13.避免在WHERE條件中使用函示，會使DB engine將資料表中要轉換的欄位全部轉成所需的Datatype才開始進行條件比對
以下代碼是作為查詢create_time欄位只需日期，而該欄位的Datatype是datetime2，儲存格式為2021-05-14 17:33:18.5100000
此舉有可能會影響到該查詢不走索引

WHERE條件函式轉換
~~~sql
SELECT TOP (1000) [id]
      ,[order_no]
      ,[order_desc]
      ,[status]
      ,[order_type]
      ,[amount]
      ,[transaction_time]
      ,[create_time]
      ,[ori_order_id]
      ,[order_amount]
FROM [pxplus_account].[dbo].[order] WITH (NOLOCK)
WHERE
    [create_time] >= '2021-05-14 00:00:00.0000000'
    AND [create_time] < '2021-05-20 00:00:00.0000000'
;
 
--執行計劃如下圖
~~~
![](https://imgur.com/XnQal45.jpg)

14.避免在WHERE條件上使用CASE WHEN
此舉依條件問題有可能會影響到該查詢不走索引，且會使取資料的邏輯變的複雜不易維護

WHERE條件使用CASE WHEN
~~~sql
SELECT TOP (1000) [id]
      ,[order_no]
      ,[order_type]
FROM [pxplus_account].[dbo].[order] WITH (NOLOCK)
WHERE
    CASE WHEN [order_type] = 1 THEN 1 ELSE 0 END = 1
;
 
--執行計劃如下圖
~~~
![](https://imgur.com/igeI6Qf.jpg)

應改為正常該有的查詢方式

WHERE不使用CASE WHEN
~~~sql
SELECT TOP (1000) [id]
      ,[order_no]
      ,[order_type]
FROM [dbo].[order] WITH (NOLOCK)
WHERE
    [order_type] = 1
;
~~~

15.使用參數化查詢，避免使用者輸入內容拼接SQL語法
EX:
SQL Injection範例
~~~sql
DECLARE @OrderNo VARCHAR(128);  --作為傳參
DECLARE @SQL VARCHAR(MAX);      --作為變數
 
--組字串方式直接執行(有SQL injection疑慮)
SET @OrderNo = ''''' OR 1=1'  -- 此段假設傳參帶入的是意圖不軌的內容
SET @SQL = '
            SELECT  [id]
                    ,[order_no]
                    ,[order_type]
            FROM [dbo].[order] WITH (NOLOCK)
            WHERE
                [order_no] = ' + @OrderNo + '
            ';
EXEC (@SQL)
 
--以下為上述SQL組語連結DB後執行的語法
SELECT  [id]
        ,[order_no]
        ,[order_type]
FROM [dbo].[order] WITH (NOLOCK)
WHERE
    [order_no] = '' OR 1=1
 
--執行後結果集如下
~~~
![](https://imgur.com/xexlCCP.jpg)
使用參數化查詢
~~~sql
DECLARE @OrderNo VARCHAR(128)   --作為傳參
SET @OrderNo = ''''' OR 1=1'    --此段假設傳參帶入的是意圖不軌的內容
 
--參數化查詢方式
SELECT  [id]
        ,[order_no]
        ,[order_type]
FROM [dbo].[order] WITH (NOLOCK)
WHERE
    [order_no] = @OrderNo
;
 
--執行後結果集如下
~~~
![](https://imgur.com/ZcDtORW.jpg)

如若需要使用組字串方式進行，一定要嚴格要求對傳參作內容轉置

SQL Injection 參數轉置
~~~sql
DECLARE @OrderNo VARCHAR(128);  --作為傳參
DECLARE @SQL VARCHAR(MAX);      --作為變數
 
--組字串方式直接執行(有SQL injection疑慮)
SET @OrderNo = ''''' OR 1=1'  -- 此段假設傳參帶入的是意圖不軌的內容
 
SET @OrderNo = REPLACE(REPLACE(REPLACE(@OrderNo,'',''),'OR',''),'1=1','')   --此段進行傳參內容轉置
 
SET @SQL = '
            SELECT  [id]
                    ,[order_no]
                    ,[order_type]
            FROM [dbo].[order] WITH (NOLOCK)
            WHERE
                [order_no] = ' + @OrderNo + '
            ';
EXEC (@SQL)
 
--以下為上述SQL組語連結DB後執行的語法
SELECT  [id]
        ,[order_no]
        ,[order_type]
FROM [dbo].[order] WITH (NOLOCK)
WHERE
    [order_no] = ''
 
 
--但具備攻擊性的關鍵字很多
--舉凡
-- ' OR 1=1
-- '; DROP TABLE XXX;
-- '; DROP DATABASE XXX;
-- '; TRUNCATE TABLE XXX;
--甚至還有更為複雜的直接建立帳號/變更密碼/賦予權限等等
--建議是交由參數化查詢代為過濾特殊字元
--.NET開發請參考
--https://docs.microsoft.com/zh-tw/aspnet/web-forms/overview/data-access/accessing-the-database-directly-from-an-aspnet-page/using-parameterized-queries-with-the-sqldatasource-cs


~~~

16.使用Join語法，需確保ON條件欄位均在索引或叢集索引上
EX:
查詢使用的SQL
~~~sql
SELECT  TOP (1000)
        B.[id] AS OrderID,
        A.[id] AS PaymentID,
        A.[order_id] AS PaymentORderID,
        A.[create_time]
FROM [dbo].[payment_order] AS A WITH (NOLOCK)
    INNER JOIN [dbo].[order] AS B ON A.[order_id] = B.[id]
WHERE
    A.[receiver_ep_account_id] = 35
;
~~~
先確認兩張表目前已有的索引
![](https://imgur.com/EGbNvKK.jpg)
確認估計執行計劃
![](https://imgur.com/EhOD4Qu.jpg)
在payment_order上採用了索引掃描

以下我們新增一個對應的索引

新增索引
~~~sql
CREATE NONCLUSTERED INDEX [IDX_test1_order_id] ON [dbo].[payment_order]
(
    [receiver_ep_account_id] ASC,
    [order_id] ASC
)
INCLUDE([create_time]) WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
GO
~~~
估計執行計劃
![](https://imgur.com/8tIepg8.jpg)

17.不在WHERE條件欄位上進行運算，有可能迫使DB engine放棄使用索引而全表掃描
EX:

對WHERE欄位做運算
~~~sql
--目的為找出TransAmount = 200的訂單編號
 
SELECT TOP (1000) [Id]
FROM [pxplus_paymentrecord].[dbo].[trans_record] WITH (NOLOCK)
WHERE
    [TransAmount] / 2 = 100.00
;
 
--執行計劃如下
~~~
![](https://imgur.com/YOSEZsJ.jpg)
建議應改為以下語法

對WHERE欄位做運算正確方式
~~~sql
--目的為找出TransAmount = 200的訂單編號
 
SELECT TOP (1000) [Id]
FROM [pxplus_paymentrecord].[dbo].[trans_record] WITH (NOLOCK)
WHERE
    [TransAmount] = 200.00
;
 
--執行計劃如下
~~~
![](https://imgur.com/MkqUmnj.jpg)

18.不在WHERE條件上使用函式
EX:
WHERE欄位上使用函式
~~~sql
SELECT [OrderNo]
FROM [pxplus_paymentrecord].[dbo].[trans_record] WITH (NOLOCK)
WHERE
    SUBSTRING([OrderNo],1,5) = 'TX902'
;
 
--以下紅框為此段執行計劃

~~~
![](https://imgur.com/wfGea8Q.jpg)

改為不在WHERE欄位上使用函式
~~~sql
SELECT [OrderNo]
FROM [pxplus_paymentrecord].[dbo].[trans_record] WITH (NOLOCK)
WHERE
    [OrderNo] LIKE 'TX902%'
;
--以下紅框為此段執行計劃
~~~
![](https://imgur.com/4ZZjsrU.jpg)
使用函式有可能造成索引掃描或式不走索引且使用函式會讓DB engine將此欄位依照需求轉成必要的資料後才進行比對
在使用了LIKE 'XXX%'之後，可以節省並降低執行成本，%放置於最後端可能可以直接使用索引搜尋

19.使用LEFT JOIN取代NOT IN / NOT EXISTS

原本NOT IN語法
~~~sql
SELECT  TOP 100
        [order_id]
        ,[payer_ep_account_id]
        ,[receiver_ep_account_id]
FROM [dbo].[payment_order] WITH (NOLOCK)
WHERE
    [payer_ep_account_id] NOT IN (SELECT [id] FROM [dbo].[ep_account])
;
~~~

盡量改用LEFT JOIN查詢(JOIN表越大效果越明顯)

LEFT JOIN取代NOT IN / NOT EXISTS
~~~sql
SELECT  TOP 100
        [order_id]
        ,[payer_ep_account_id]
        ,[receiver_ep_account_id]
FROM [dbo].[payment_order] AS A WITH (NOLOCK)
    LEFT JOIN [dbo].[ep_account] AS B WITH (NOLOCK)
        ON A.[payer_ep_account_id] = B.[id]
WHERE
    B.[id] IS NULL
;
~~~

20.更新大量筆數資料，不應大範圍直接下達更新，建議已批次更新每次小量小量更新
也避免因新增/刪除/修改大量資料導致Lock升級至Table Lock，通常比較容易理解的是超過5000筆數據同時被更新一定會升級為Table Lock
也可以參考網址說明:
https://learn.microsoft.com/zh-tw/troubleshoot/sql/performance/resolve-blocking-problems-caused-lock-escalation#lock-escalation-thresholds

EX:
批次更新
~~~sql
--假設以下資料表每天都有非常大量的資料寫入
--在訊息顯示上，每天都很大量，而我們需要使用SQL去進行更新[IsDisable]欄位值，統一一律改成1
--一般常見執行如下  指定時間點/區間 然後將該表該欄位進行更新
 
UPDATE [pxplus_paymentrecord]
SET [IsDisable] = 1
WHERE
    [CreateTime] >= '2022/03/01'
    AND [IsDisable] = 0
;
 
--應該將語法改為下面方式 遞迴執行
WHILE(SELECT 1 FROM [pxplus_paymentrecord] WITH (NOLOCK) WHERE [CreateTime] >= '2022/03/01' AND [IsDisable] = 0)
BEGIN
    UPDATE TOP (3000) [pxplus_paymentrecord]
    SET [IsDisable] = 1
    WHERE
        [CreateTime] >= '2022/03/01'
        AND [IsDisable] = 0
    ;
 
    WAITFORDELAY '00:00:01'
END
~~~

21.
禁止使用SELECT * 的方式取資料，應該明確指定需要的欄位，避免資源浪費與不必要的IO消耗
SELECT解析過程如下
FROM => ON => JOIN => WHERE => GROUP BY => WITH CUBE或WITH ROLLUP => HAVING => SELECT => ORDER BY

![](https://imgur.com/wSo58ai.jpg)
執行的SELECT *那段，此時DB Engine會在磁碟上取得所有欄位資料，
而資料寫入因頻繁的INSERT/DELETE/UPDATE致使新的一筆資料INSERT在磁區上不連續空間中
SELECT導致DB Engine必須在取得所有欄位資料後才能顯示結果集，導致效能上佔有些許消耗
只SELECT必要的欄位是能增加速度的方式

22.使用了#或##開頭的暫存表，應該在Session結束前主動TRUNCATE TABLE 及 DROP TABLE，避免資源浪費，@或@@暫存表用在記憶體上則不在此限(##及@@代表全域暫存表)

23.必須使用暫存表暫存資料時，除@或@@暫存表之外，在確定筆數為大量時必須使用SELECT INTO的方式而不使用INSERT INTO，資料量較小時則不限制

24.禁止使用CURSOR語法，在尚未精通SQL之前使用CURSOR語法容易引發blocking，與Cursor中查詢的資料表大小有關

25.止使用Trigger語法，在資料庫管理上很難追查

26.除非必要否則不大資料表上使用ORDER BY / GROUP BY / HAVING，此舉會增加額外成本

27.索引命名，依照以下類型後面加上 _TableName_ColumnName
|類型|用途|
|-|-|
|PK	叢集索引/主鍵|
|FK	|外部鍵|
|IX|一般索引|
|UX|唯一索引|

28.索引設計
可參考 https://docs.microsoft.com/zh-tw/sql/relational-databases/sql-server-index-design-guide?view=sql-server-ver15

A. 盡量以複合式索引設計，將WHERE條件中可能用到的列為索引中的欄位，將SELECT中所包含的欄位列為索引中包含的欄位(include Column)
B. 原則上盡量別使用經常更新的欄位作為索引欄位，假設訂單狀態欄位經常被更新，則不太適合作為索引尤其是叢集索引(Primary Key)
C. 索引的欄位盡量以明確分類一定數量的內容做為參考，例如0/1、男/女這類型其實不太適合
D. 每張表盡量別超過5張索引
E. 盡量別使用允許Null的欄位作為索引
F. text, ntext, varchar(max), nvarchar(max) 別作為索引欄位
G. 叢集索引(或Primary Key)的多欄位設計，盡量確保以升序方式做為考量，避免經常性中間寫入鍵值導致經常性分頁
H. 越常被使用到的欄位，其在索引中的位置應於最左(或最上)，經常被使用在WHERE條件中(= > < Bwtween)也盡量放在最左邊
I. 索引欄位的升序降序規則，依照較經常被ORDER BY使用的欄位做設計

29.使用SQL Profiler即時取得執行語句
![](https://imgur.com/PLFeU1Y.jpg)
![](https://imgur.com/F8RVBjz.jpg)
![](https://imgur.com/G3Onjq2.jpg)

上述事件僅需依照所需查看的內容勾選即可，執行結果如下

![](https://imgur.com/xBZ3m02.jpg)
此工具不得任意在Production環境上使用，將會側錄所有內容並拖慢DB執行效率

30.Bitwise用法

建立測試資料
~~~sql
- 建立暫存資料
DECLARE @a INT
CREATE TABLE #T (t1 INT)
INSERT INTO #T VALUES (23)
 
DECLARE @a INT
SET @a = 8
SELECT t1 FROM #T WHERE (t1 & @a) = @a
-- 結果為NULL，23的二進制表示並未包含8的二進制表示所有標示為1位址
-- 23的二進制表示 ０００１　０１１１
--  8的二進制表示 ００００　１０００
 
DECLARE @a INT
SET @a = 4
SELECT t1 FROM #T WHERE (t1 & @a) = @a
-- 結果為23，23的二進制表示包含4的二進制表示所有標示為1位址
-- 23的二進制表示 ０００１　０１１１
--  4的二進制表示 ００００　０１００
 
DECLARE @a INT
SET @a = 25
SELECT (t1 & @a) FROM #T WHERE (t1 & @a) = @a
-- 23的二進制表示 ０００１　０１１１
-- 25的二進制表示 ０００１　１００１
-- 若要做為權限或資格控管的用法
-- 如果要查詢欄位t1是否包含了變數值25的二進制表示上所有標示為1位址，則只列出有符合的資料列，此處只有一筆資料為23並且列為不符合(以(t1 & @a)=@a為例)
 
DECLARE @a INT
SET @a = 25
SELECT (t1 & @a) FROM #T WHERE (t1 & @a) > 0
-- 如果要查詢欄位t1與變數值25的二進制表示上所有標示為1位址，則在16、1的bit上符合(以(t1 & @a)>0為例)，並回傳加總後的17

~~~

