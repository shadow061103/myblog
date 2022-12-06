---
title: "mysql的row_number跟partition用法"
date: 2022-12-06T16:30:21+08:00
draft: false
tags: ["DB"]
type: post
author: "Steve Lin"
description: ""
---

情境:
A表會對應B表多個交易
需要加總A表金額(B表算完的)並抓B表的第一筆的欄位名稱做顯示

原始資料表
![](https://imgur.com/2FPV8E3.jpg)


### 使用row_number對資料作編號
ROW_NUMBER() OVER (<partition_definition> <order_definition>) 
#### 單純用order by
~~~sql
SELECT *,
    ROW_NUMBER() OVER(ORDER BY id) AS row_numb
FROM exam_results;
~~~
![](https://imgur.com/fOvsria.jpg)

#### 單純用partition(對資料分割，類似group by分組)
~~~sql
SELECT *,
    ROW_NUMBER() OVER(PARTITION BY year) AS row_numb
FROM exam_results;
~~~
![](https://imgur.com/FhUjQ0k.jpg)

#### 組合使用
對年份分群並排序result
~~~sql
 SELECT *,
    ROW_NUMBER() OVER(PARTITION BY year ORDER BY result DESC) AS RowNumber
  FROM exam_results;
~~~
![](https://imgur.com/T6cKDsD.jpg)



### 取分群資料的第一筆
~~~sql
WITH added_row_number AS (
  SELECT
    *,
    ROW_NUMBER() OVER(PARTITION BY year ORDER BY result DESC) AS RowNumber
  FROM exam_results
)
SELECT
  *
FROM added_row_number
WHERE RowNumber = 1;

~~~
![](https://imgur.com/3qfi84n.jpg)

### 提供兩種解法，也可以使用window function，這邊欄位跟DB名稱都有調整過名稱(非正式使用)

group by 
缺點是用max()或min()取得顯示欄位，有可能會不一樣
~~~sql
select 
r.code,
r.name,
r.brand_name,
min(m.store_name),
sum(r.amount),
sum(r.fee)
from payment as r
join trade as m 
on m.id = (
select id
from trade
where r.number=number 
order by id desc limit 1
)
where r.create_time between '2022-11-01 00:00:00' and '2022-11-30 23:59:59'
and r.code in ('123456')
and r.status between 1 and 4
group by r.code,r.name,r.brand_name;
~~~

使用window funciton的FIRST_VALUE取資料，搭配sum()聚合函數
~~~sql
WITH result AS (
select 
 FIRST_VALUE(r.code)  OVER w AS 'code',
 FIRST_VALUE(r.name)  OVER w AS 'name',
 FIRST_VALUE(r.brand_name)  OVER w AS 'brand_name',
FIRST_VALUE(m.store_name) OVER w AS 'store_name',
sum(r.amount) OVER w AS 'amount',
sum(r.fee) OVER w AS 'fee',
ROW_NUMBER() OVER w AS RowNumber
from payment as r
join trade as m 
on m.id = (
select id
from trade
where r.number=number 
order by id desc limit 1
)
where r.create_time between '2022-11-01 00:00:00' and '2022-11-30 23:59:59'
and r.code ='123456'
and r.status between 1 and 4
WINDOW w AS (PARTITION BY r.code ORDER BY r.code DESC)
)
SELECT
  *
FROM result
WHERE RowNumber = 1;
~~~

### 參考
[How to Select the First Row in Each GROUP BY Group](https://learnsql.com/cookbook/how-to-select-the-first-row-in-each-group-by-group/)

[語法教學](https://www.begtut.com/mysql/mysql-row-number-function.html)

[MySQL 中 Row_Number() 函式的使用](https://www.delftstack.com/zh-tw/howto/mysql/row_number-in-mysql/)

[Window Function Descriptions](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_first-value)

[Window Function Concepts and Syntax](https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html)