---
layout: post
title: In-Memory Column Store
category: [oracle]
tags: [in-memory]
---
##### In-Memory Column

###### In-Memory Column Store特性
----
从12.1.0.2版本开始，新增了In-Memory Column Store (IM column store)特性。

该特性开启后会在数据库启动阶段在SGA中分配一块静态的内存池In-Memory Area，用于存放以列式存储的用户表。

数据在内存的独立区域中按照列式存储，数据是被压缩存放的，内存与列式压缩可以极大提升查询的性能。

如下图所示：
![image](/img/2021-02-23-oracle-inm/inm_1.png)

###### 行式和列式数据库
----
常见的行式数据库系统有：MySQL、Postgres和MS SQL Server。

常见的列式数据库有： Vertica、 Paraccel (Actian Matrix，Amazon Redshift)、 Sybase IQ、 Exasol、 Infobright、 InfiniDB、 MonetDB (VectorWise， Actian Vector)、 LucidDB、 SAP HANA、 Google Dremel、 Google PowerDrill、 Druid、 kdb+。

不同的数据存储方式适用不同的业务场景，数据访问的场景包括：进行了何种查询、多久查询一次以及各类查询的比例；每种类型的查询(行、列和字节)读取多少数据；读取数据和更新之间的关系；使用的数据集大小以及如何使用本地的数据集；是否使用事务,以及它们是如何进行隔离的；数据的复制机制与数据的完整性要求；每种类型的查询要求的延迟与吞吐量等等。

###### OLAP场景的关键特征
----
1. 绝大多数是读请求
2. 数据以相当大的批次(> 1000行)更新，而不是单行更新;或者根本没有更新。
3. 已添加到数据库的数据不能修改。
4. 对于读取，从数据库中提取相当多的行，但只提取列的一小部分。
5. 宽表，即每个表包含着大量的列
6. 查询相对较少(通常每台服务器每秒查询数百次或更少)
7. 对于简单查询，允许延迟大约50毫秒
8. 列中的数据相对较小：数字和短字符串(例如，每个URL 60个字节)
9. 处理单个查询时需要高吞吐量(每台服务器每秒可达数十亿行)
10. 事务不是必须的
11. 对数据一致性要求低
12. 每个查询有一个大表。除了他以外，其他的都很小。
13. 查询结果明显小于源数据。换句话说，数据经过过滤或聚合，因此结果适合于单个服务器的RAM中



> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
