---
layout: post
title: New 12c Feature:Online Statistics Gathering for Bulk Loads
category: [oracle]
tags: [_optimizer_gather_stats_on_load]
---
##### 批量数据加载的在线统计信息收集

###### 批量数据加载
----
12c版本之前我们的数据库在进行批量加载操作时，例如CTAS操作后，往往需要重新收集新表的统计信息。

在Oracle Database 12c中，如下两种Bulk-Load方式下，系统将会自动收集表上的统计信息。
1. CTAS – Create Table As Select …
2. IIS – Insert Into … Select …

如何查询版本及数据库配置是否支持这个新特性，可以通过以下语句查询：

```
SQL> SELECT i.ksppinm  NAME,
  2         i.ksppity  TYPE,
  3         v.ksppstvl VALUE,
  4         v.ksppstdf isdefault
  5    FROM x$ksppi i, x$ksppcv v
  6   WHERE i.indx = v.indx
  7     AND i.ksppinm LIKE '/_%optimizer_gather_stats_on_load%' ESCAPE '/';

NAME                                                                                   TYPE VALUE                                                                            ISDEFAULT
-------------------------------------------------------------------------------- ---------- -------------------------------------------------------------------------------- ---------
_optimizer_gather_stats_on_load                                                           1 TRUE                                                                             TRUE

```

###### NO_GATHER_OPTIMIZER_STATISTICS
----
除了通过参数控制该特性，还可以使用NO_GATHER_OPTIMIZER_STATISTICS这个hint来控制具体SQL。

###### 受限情况
1. Index statistics or histograms. If those are required the must be gathered in a separate operation. The following call will gather missing statistics, but will not re-gather table or column statistics unless the existing statistics are already stale.
2. Non-empty segments, as described above.
3. Tables in built-in schemas. Only those in user-defined schemas.
4. Nested, index-organized or external tables.
5. Global temporary tables using the ON COMMIT DELETE ROWS clause.
6. Table with virtual columns.
7. Tables if the PUBLISH preference is set to FALSE for DBMS_STATS.
8. Tables with locked statistics.
9. Partitioned tables using incremental statistics, where the insert is not explicitly referencing a partition using the PARTITION clause.
10. Tables loaded using multitable inserts.

比如在sys用户下执行CTAS就是无法自动收集统计信息的
```
SQL> select owner, table_name, sample_size, last_analyzed, stale_stats
  2    from dba_tab_statistics
  3   where table_name = 'T_APPROXTEST';

OWNER                                                                            TABLE_NAME                                                                       SAMPLE_SIZE LAST_ANALYZED STALE_STATS
-------------------------------------------------------------------------------- -------------------------------------------------------------------------------- ----------- ------------- -----------
SYS                                                                              T_APPROXTEST                                                                                               
C##TEST                                                                          T_APPROXTEST                                                                        11665280 2021-02-26 17 NO

```






> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
