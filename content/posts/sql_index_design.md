---
title: "資料表與索引設計"
date: 2022-11-25T15:52:04+08:00
draft: false
tags: ["DB"]
type: post
author: "Steve Lin"
description: ""
---
資料表依照需求及未來可能使用的欄位設計
基於一般OLTP資料庫經常得混搭OLAP報表及分析使用，除程式的Class設計資料表欄位，但需考量非交易狀態時的其他應用而新增其他欄位，
舉例來說，訂單表結構化的資料欄位本就不少，但訂單經常會被分天或分小時使用，則可以在資料表上增添一個年月日的數字欄位並加入索引，
在查詢上則可以限制查詢的WHERE條件一定要有年月日(數字)欄位，或若有其他可分組的數據，也可增加對應的分組欄位，以下使用訂單日期
做範例

EX:
原有的訂單表
~~~sql
CREATE TABLE [dbo].[payment_order](
    [id] [bigint] IDENTITY(1,1) NOT NULL,
    [order_id] [bigint] NOT NULL,
    [payer_ep_account_id] [bigint] NOT NULL,
    [receiver_ep_account_id] [bigint] NOT NULL,
    [from_collection_amount] [decimal](9, 2) NOT NULL,
    [from_deposit_amount] [decimal](9, 2) NOT NULL,
    [remaining_collection_refund_amount] [decimal](9, 2) NOT NULL,
    [remaining_deposit_refund_amount] [decimal](9, 2) NOT NULL,
    [modify_time] [datetime2](7) NOT NULL,
    [create_time] [datetime2](7) NOT NULL,
 CONSTRAINT [PK__payment___3213E83F612B5E16] PRIMARY KEY CLUSTERED
(
    [id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]
GO
 
ALTER TABLE [dbo].[payment_order] ADD  CONSTRAINT [DF__payment_o__order__65370702]  DEFAULT (NULL) FOR [order_id]
GO
 
ALTER TABLE [dbo].[payment_order] ADD  CONSTRAINT [DF__payment_o__payer__662B2B3B]  DEFAULT (NULL) FOR [payer_ep_account_id]
GO
 
ALTER TABLE [dbo].[payment_order] ADD  CONSTRAINT [DF__payment_o__recei__671F4F74]  DEFAULT (NULL) FOR [receiver_ep_account_id]
GO
 
CREATE NONCLUSTERED INDEX [IX_payment_order_order_id] ON [dbo].[payment_order]
(
    [order_id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
GO
~~~
![](https://imgur.com/6jpW6Io.jpg)
調整的訂單表
~~~sql
CREATE TABLE [dbo].[payment_order_test](
    [id] [bigint] IDENTITY(1,1) NOT NULL,
    [order_id] [bigint] NULL,
    [cdate] [int] NULL,
    [mdate] [int] NULL,
    [payer_ep_account_id] [bigint] NULL,
    [receiver_ep_account_id] [bigint] NULL,
    [from_collection_amount] [decimal](9, 2) NOT NULL,
    [from_deposit_amount] [decimal](9, 2) NOT NULL,
    [remaining_collection_refund_amount] [decimal](9, 2) NOT NULL,
    [remaining_deposit_refund_amount] [decimal](9, 2) NOT NULL,
    [modify_time] [datetime2](7) NOT NULL,
    [create_time] [datetime2](7) NOT NULL,
 CONSTRAINT [PK_payment_order_test] PRIMARY KEY CLUSTERED
(
    [id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]
GO
 
ALTER TABLE [dbo].[payment_order_test] ADD  CONSTRAINT [DF_payment_order_test_order_id]  DEFAULT (NULL) FOR [order_id]
GO
 
ALTER TABLE [dbo].[payment_order_test] ADD  CONSTRAINT [DF_payment_order_test_payer_ep_account_id]  DEFAULT (NULL) FOR [payer_ep_account_id]
GO
 
ALTER TABLE [dbo].[payment_order_test] ADD  CONSTRAINT [DF_payment_order_test_receiver_ep_account_id]  DEFAULT (NULL) FOR [receiver_ep_account_id]
GO
 
CREATE NONCLUSTERED INDEX [IDX_cdates] ON [dbo].[payment_order_test]
(
    [cdate] ASC,
    [mdate] ASC
)
INCLUDE([order_id]) WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
GO
 
CREATE NONCLUSTERED INDEX [IDX_epaccount_receiveraccount] ON [dbo].[payment_order_test]
(
    [payer_ep_account_id] ASC,
    [receiver_ep_account_id] ASC
)
INCLUDE([order_id],[cdate],[mdate],[from_collection_amount],[from_deposit_amount],[remaining_collection_refund_amount],[remaining_deposit_refund_amount],[modify_time],[create_time]) WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
GO
 
CREATE NONCLUSTERED INDEX [IDX_order_id] ON [dbo].[payment_order_test]
(
    [order_id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
GO
~~~
![](https://imgur.com/TlZ6YiY.jpg)

以下兩表比較

原訂單表查詢特定日期
~~~sql
SELECT  [order_id],
        [payer_ep_account_id],
        [receiver_ep_account_id],
        [from_collection_amount],
        [from_deposit_amount],
        [remaining_collection_refund_amount],
        [remaining_deposit_refund_amount]
FROM [dbo].[payment_order] AS a
WHERE
    [create_time] >= '2022-03-17 00:00:00.000'
    AND [create_time] < '2022-03-18 00:00:00.000'
~~~
![](https://imgur.com/FvrfqPr.jpg)

新訂單表查詢日期
~~~sql
SELECT  [order_id],
        [payer_ep_account_id],
        [receiver_ep_account_id],
        [from_collection_amount],
        [from_deposit_amount],
        [remaining_collection_refund_amount],
        [remaining_deposit_refund_amount]
FROM [dbo].[payment_order_test] AS a
WHERE
    [cdate] >= 20220317
    AND [cdate] < 20220318
~~~

![](https://imgur.com/Qh3MlMQ.jpg)

外部鍵(Foreign key)謹慎使用，由於Foreign key在判斷時是會另開session執行，在高併發交易時DB主機容易有控制外的連線數量，
另一點是可能引發Deadlock情況發生，以下模擬兩張資料表使用了FK如何產生Deadlock
EX:

建立範例資料表及資料
~~~sql
CREATE TABLE Tbl1 (id INT NOT NULL PRIMARY KEY CLUSTERED, col INT)
 
CREATE TABLE Tbl2 (id INT NOT NULL PRIMARY KEY CLUSTERED, col INT REFERENCES dbo.Tbl1(id))

~~~

接著開啟一個查詢分頁執行Session 1

Session 1
~~~sql
BEGIN TRAN
    INSERT dbo.Tbl1 (id, col) VALUES (2, 999)
~~~

開啟另一個查詢分頁執行Session 2
Session 2
~~~sql
BEGIN TRAN
    INSERT dbo.Tbl1 (id, col) VALUES (3, 100)
~~~

此時Session 1及Session 2應該均為Transaction為完成也沒有錯誤發生的情況，目前因Transaction尚未完成而處於Blocking狀態
由Session 1中新增一個查詢(整個Transaction的一個部分)，並未發生Deadlock狀態

引發交叉寫入
~~~sql
INSERT dbo.Tbl2 (id, col) VALUES (111, 555)
~~~

接著Session 2中心新增一個查詢，此時引發了以下Deadlock狀況

交叉鎖定引發ForeignKey及Deadlock錯誤
~~~sql
INSERT dbo.Tbl2 (id, col) VALUES (111, 2)
~~~

![](https://imgur.com/5cXim0t.jpg)
接著檢查Session 1中顯示了Foreign Key的錯誤
![](https://imgur.com/sloAd3U.jpg)

Datatype定義需明確理解可能範圍
如數字類型，數字不會超過255或正負值127之間則不使用smallint類型，數字不會超過65535或正負值32767則不使用int類型，數字類型的大小會影響實際儲存空間
請參考以下
![](https://imgur.com/NtUONjP.jpg)
如字串類型，可以定長就使用CHAR或NCHAR，若無法肯定才使用VARCHAR或NVARCHAR，
請參考已下
![](https://imgur.com/6JmjMdD.jpg)
![](https://imgur.com/Xroxf4N.jpg)

若有超過長度900之字串欄位則無法有效添加索引，需要達成搜尋條件需利用全文檢索引擎或是其他拆字方案，下圖顯示超過特定長度之欄位無法添加索引

建立範例表
~~~sql
CREATE TABLE [dbo].[Tbl3](
    [id] [int] NOT NULL,
    [content] [varchar](max) NULL,
 CONSTRAINT [PK_Tbl3] PRIMARY KEY CLUSTERED
(
    [id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
~~~
![](https://imgur.com/Ye4DrGm.jpg)

- 欄位盡量不要允許NULL，最佳建議是給予每個欄位一個預設值
NULL在SQL三值邏輯(TRUE/FALSE/UNKNOWN)中屬於UNKNOWN

可參考 https://docs.microsoft.com/zh-tw/sql/t-sql/language-elements/null-and-unknown-transact-sql?view=sql-server-ver15
欄位允許NULL時，SQL Server會使用bitmap來標註該欄位是否為允許NULL，對於執行SQL會有額外解碼bitmap的成本

可參考 https://dotblogs.com.tw/ricochen/2012/01/30/67092
- 資料表名稱長度最大限制，最好不要太長的命名，淺顯易懂即可，並且不能與其他物件或類型同名
SQL Server : 128字元
MySQL : 64字元
Oracle : 30字元

- 資料表定序需與資料庫定序一致，盡量避免不同定序產生
SQL Server : Chinese_Taiwan_Stroke_CI_AS
MySQL : utf8mb4_0900_ai_ci  (建議使用 utf8mb4_general_ci)
CI 指定不區分大小寫，CS 指定區分大小寫
AI 指定不區分腔調字，AS 指定區分腔調字
- 正規化設計
建議最多到2NF
正規化說明可參考
https://rileylin91.github.io/2020/05/21/MSSQL-6-Normalization/

- Primary Key設計
在SQL Server的Primary Key設計上比較常見的是與叢集索引一起設定，但也有叢集索引及Primary Key是分開的例子
MySQL則是Primark Key一定是叢集索引
1. 每張資料表只能有一個Primary Key(叢集索引)而可以有多個Non Clustered Index
2. Primary Key一定是升序排序且具備唯一性質及非NULL限制
3. 可以使用單一欄位或多欄位建立，但需維持由左至右或由上至下的欄位順序可能存放的值，是符合2所述條件
    可避免資料寫入時造成分頁
4. 叢集索引比非叢集索引速度更快
5. 資料儲存在叢集索引上，以SQL Server可以將叢集索引建立在不同的File Group時，則實體資料使用的空間歸屬在File Group所在的磁碟上
6. 盡量不使用uniqueidentifier來作為Primary Key
![](https://imgur.com/fDEUaQw.jpg)
- Index設計
1. 參考資料表中的數據，備經常使用的是哪些欄位做複合式索引，並且排序依照經常被查詢使用的方式排序，可以免去查詢與法的指定排序
2. 經常被使用在範圍搜尋時使用非叢集索引，並且依照B+Tree的結構選擇適合分組的欄位，依照資料數量與選定的欄位值的可能種類用以下公式約略計算
    資料表中資料總數量 / 欄位值可能的分類 = 參考值 (其參考值越接近1越好)
    EX:
        在擁有1000筆的數據中需要分類男女性別並套用公式
        1000 / 2 = 0.002 (不建議使用)
3. 非叢集索引數量加上叢集索引最多不要超過6個，會影響寫入資料時的效能問題
4. SQL Server非叢集索引設計上，若能以空間換取時間，則盡量把可能使用到的欄位都加進包含的資料行
5. 如果該表經常會與其他資料表做Join，則建議索引設計上需要將最經常被使用到的欄位放在越前面，且ON與WHERE條件的欄位都要列入考量