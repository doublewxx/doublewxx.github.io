---
title: 工作中遇到的数据库查询问题
date: 2021-06-30 10:47:30
tags: 
  - database
categories: 数据库
---
### 问题描述
我们有一张表，表格如下
| ID  | pubID | type | modifyTime          |
| --- | ----- |
| 1   | 80    | 2    | 2021-06-28 17:33:43 |
| 2   | 80    | 2    | 2021-06-28 17:35:43 |
| 3   | 79    | 2    | 2021-06-28 17:45:43 |
| 4   | 80    | 1    | 2021-06-28 17:55:43 |
### 过程记录
需求：传入pubID数组，希望查询出来每个pubID最近一次修改的记录
数据库命令1：
```javascript
select * from Table where pubID in (80, 79) and type=2 group by pubID order by modifyTime DESC;
```
数据库命令2：
```javascript
select * from Table where pubID in (80, 79) and type=2 group by pubID;
```
这两个命令返回结果一样，主要是因为mysql的查询步骤，https://juejin.cn/post/6844903967210602509     
参考这个文档，先进行了groupby之后就只有一条数据，orderby不起作用
数据库命令3：
```javascript
select *,max(modifyTime) from Table where pubID in (80, 79) and type=2 group by pubID;
```
返回的是第一条数据，但是modifyTime是最大值的那个，并非第一条记录的。由group by 分组后显示的是第一条记录，而max()取的是相同pubID中的最大modifyTime值。
### 解决方案
子查询
```javascript
const result = await Database
.rawQuery(`select a.* from ${config.table} a join(select pubID, max(modifyTime) as mtime from ${config.table} where type = ${changeType} and pubID in (${pubIDlist}) group by pubID) as b on a.pubID = b.pubID and a.modifyTime = b.mtime;`);
```
