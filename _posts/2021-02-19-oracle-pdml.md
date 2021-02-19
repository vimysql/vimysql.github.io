---
layout: post
title: New 12c Hint:ENABLE_PARALLEL_DML Lets you Easily Enable Parallel DML (PDML) at the Statement Level (Doc ID 1991034.1)
category: [oracle]
tags: [PDML]
---

##### PDML新hint的版本说明
只有满足以下版本要求，才能使用该新hint
Oracle Database - Enterprise Edition - Version 12.1.0.1 and later

###### 官方原文解释如下
----
Parallel DML (PDML) must be explicitly enabled in order for DML to be considered for parallel execution.  In RDBMS versions lower than 12c, you could accomplish this only at the session level by using an ALTER SESSION statement. Assuming restrictions on parallel DML do not apply to your situation, once the session is altered, all further statements in the session will be candidates for execution of DML in parallel.

###### 以下为在11.2.0.4和在19C中的对比
----
11.2.0.4版本中如下所示，hint中查询不到有ENABLE_PARALLEL_DML新hint特性
```
SQL> select a.BANNER,
  2         (select b.NAME from v$sql_hint b where b.name like 'ENABLE%') hint
  3    from v$version a;

BANNER                                                                           HINT
-------------------------------------------------------------------------------- ----------------------------------------------------------------
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production     
PL/SQL Release 11.2.0.4.0 - Production                                           
CORE	11.2.0.4.0	Production                                                         
TNS for 64-bit Windows: Version 11.2.0.4.0 - Production                          
NLSRTL Version 11.2.0.4.0 - Production     
```

19C中查询情况如下，版本高于12.1.0.1，可以查询到ENABLE_PARALLEL_DML
```
SQL> select a.BANNER,
  2         (select b.NAME from v$sql_hint b where b.name like 'ENABLE%') hint
  3    from v$version a;

BANNER                                                                           HINT
-------------------------------------------------------------------------------- ----------------------------------------------------------------
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production           ENABLE_PARALLEL_DML

```

###### 19C中演示使用ENABLE_PARALLEL_DML的效果
----
创建测试表t_pdml，并对其pdml插入数据
```
create table t_pdml as select * from dba_tables where 1=0;
insert into t_pdml select * from dba_tables;
commit;
explain plan for 
insert /*+parallel(8) ENABLE_PARALLEL_DML */
into t_pdml 
  select * from t_pdml;
select * from table(dbms_xplan.display());

PLAN_TABLE_OUTPUT
Plan hash value: 1152776488
 
----------------------------------------------------------------------------------------------------------------------------
| Id  | Operation                          | Name     | Rows  | Bytes | Cost (%CPU)| Time     |    TQ  |IN-OUT| PQ Distrib |
----------------------------------------------------------------------------------------------------------------------------
|   0 | INSERT STATEMENT                   |          |     1 |  1473 |     2   (0)| 00:00:01 |        |      |            |
|   1 |  PX COORDINATOR                    |          |       |       |            |          |        |      |            |
|   2 |   PX SEND QC (RANDOM)              | :TQ10000 |     1 |  1473 |     2   (0)| 00:00:01 |  Q1,00 | P->S | QC (RAND)  |
|   3 |    LOAD AS SELECT (HYBRID TSM/HWMB)| T_PDML   |       |       |            |          |  Q1,00 | PCWP |            |
|   4 |     OPTIMIZER STATISTICS GATHERING |          |     1 |  1473 |     2   (0)| 00:00:01 |  Q1,00 | PCWP |            |
|   5 |      PX BLOCK ITERATOR             |          |     1 |  1473 |     2   (0)| 00:00:01 |  Q1,00 | PCWC |            |
|   6 |       TABLE ACCESS FULL            | T_PDML   |     1 |  1473 |     2   (0)| 00:00:01 |  Q1,00 | PCWP |            |
----------------------------------------------------------------------------------------------------------------------------
 
Note
-----
   - Degree of Parallelism is 8 because of hint

```
除了上面使用的hint方式，也可以在会话级别启用pdml，可通过以下命令启用
```
ALTER SESSION ENABLE PARALLEL DML;
<execute DML statement> 
```

###### ENABLE_PARALLEL_DML的受限情形
----
即使在19C等可使用ENABLE_PARALLEL_DML这个新hint的环境中，也是存在受限情形的。如果DML操作的对象表上存在触发器，那么这种情况下是无法启用PDML的。
如下所示：
![image](/img/2021-02-19-oracle-pdml/pdml_1.png)

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
