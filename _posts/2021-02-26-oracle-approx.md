---
layout: post
title: New 12c Feature:Approximate Count Distinct
category: [oracle]
tags: [approx]
---
##### Approximate Count Distinct

###### APPROX_COUNT_DISTINCT()
----
Oracle提供了一个新的优化的sql函数APPROX_COUNT_DISTINCT()，这个函数能够帮助我们去计算一个近似的去重的总数。

下面我们创建一个测试表，测试下这个函数的效果。

创建测试表：
```
create table t_approxtest 
as select * from dba_objects;
insert into t_approxtest
  select * from t_approxtest;
insert into t_approxtest
  select * from t_approxtest;
......#重复多次插入操作
commit;
```

测试语句如下，效果还是很明显：
```
SQL> set time on
15:50:46 SQL> set timing on 
15:50:51 SQL> select count(*) from t_approxtest;

  COUNT(*)
----------
  11665280
Executed in 0.308 seconds

15:50:53 SQL> select count(distinct(object_id)) from t_approxtest;

COUNT(DISTINCT(OBJECT_ID))
--------------------------
                     91134
Executed in 2.42 seconds

15:51:03 SQL> select count(distinct(object_id)) from t_approxtest;

COUNT(DISTINCT(OBJECT_ID))
--------------------------
                     91134
Executed in 1.758 seconds

15:51:06 SQL> select count(distinct(object_id)) from t_approxtest;

COUNT(DISTINCT(OBJECT_ID))
--------------------------
                     91134
Executed in 1.781 seconds

15:51:11 SQL> select APPROX_COUNT_DISTINCT(object_id) from t_approxtest;

APPROX_COUNT_DISTINCT(OBJECT_I
------------------------------
                         92772
Executed in 0.863 seconds

15:52:04 SQL> select APPROX_COUNT_DISTINCT(object_id) from t_approxtest;

APPROX_COUNT_DISTINCT(OBJECT_I
------------------------------
                         92772
Executed in 0.785 seconds
```

###### 适合使用的场景
----
1. OLAP数仓环境。
2. 需要统计的去重数据所在列不存在索引的情况下。
3. 仅仅为了统计近似去重值，如果是需要获取准确去重值，慎用。

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
