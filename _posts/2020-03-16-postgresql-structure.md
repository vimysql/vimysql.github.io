---
layout: post
title: 'PostgreSQL structure of a database cluster（Part I）'
date: '2020-03-16'
header-img: "img/post-bg-android.jpg"
tags:
     - postgresql
author: '王小胖'
---

# Structure of a database cluster
 
## Logical Structure of Database Cluster
----
下图展示了一个database cluster的逻辑结构，每个database是数据库对象的集合。

在PostgreSQL中，databases也是数据库对象，并且它们各自都是逻辑独立分开的。所有其他数据库对象（例如，表，索引，其他）都属于它们各自的数据库。

![image](./img/2020-03-16-postgresql-structure/structure_1.png)

PostgreSQL中的所有数据库对象，在数据库内部都由各自的对象标识管理（OIDs），OID是无符号的4字节整数。

数据库对象和OIDs之前的关系存放在对应的系统目录中（system catalogs），根据对象类型分类。例如，数据库和堆表的OIDs依次存放在pg_database和pg_class中，因此你可以通过如下查询找到你想要的OIDs。

```
wz=# SELECT datname, oid FROM pg_database WHERE datname = 'wz';
 datname |  oid  
---------+-------
 wz      | 16384
(1 row)

wz=# SELECT relname, oid FROM pg_class WHERE relname = 'test';
 relname |  oid  
---------+-------
 test    | 16385
(1 row)

```
## Physical Structure of Database Cluster
----
一个database cluster大体上是一个目录（base directory），并且它包含一些子目录和很多文件。如果你执行initdb来初始化一个新database cluster，那么在指定目录下将创建一个base directory。虽然不是必须的，base directory路径通常设置为PGDATA环境变量的对应值。

下图展示PostgreSQL的物理结构。虽然PostgreSQL支持tablespaces，但是这个术语的含义与其他RDBMS不同。PostgreSQL中的tablespace是在base directory之外包含某些数据的一个directory。

![image](img/2020-03-16-postgresql-structure/structure_2.png)

### Layout of a Database Cluster

|	files	|	description	|
|	---	|	---	|
|	PG_VERSION	|	A file containing the major version number of PostgreSQL	|
|	pg_hba.conf	|	A file to control PosgreSQL's client authentication	|
|	pg_ident.conf	|	A file to control PostgreSQL's user name mapping	|
|	postgresql.conf	|	A file to set configuration parameters	|
|	postgresql.auto.conf	|	A file used for storing configuration parameters that are set in ALTER SYSTEM (version 9.4 or later)	|
|	postmaster.opts	|	A file recording the command line options the server was last started with	|

|	subdirectories	|	description	|
|	---	|	---	|
|	base/	|	Subdirectory containing per-database subdirectories.	|
|	global/	|	Subdirectory containing cluster-wide tables, such as pg_database and pg_control.	|
|	pg_commit_ts/	|	Subdirectory containing transaction commit timestamp data. Version 9.5 or later.	|
|	pg_clog/ (Version 9.6 or earlier)	|	Subdirectory containing transaction commit state data. It is renamed to pg_xact in Version 10.	|
|	pg_dynshmem/	|	Subdirectory containing files used by the dynamic shared memory subsystem. Version 9.4 or later.	|
|	pg_logical/	|	Subdirectory containing status data for logical decoding. Version 9.4 or later.	|
|	pg_multixact/	|	Subdirectory containing multitransaction status data (used for shared row locks)	|
|	pg_notify/	|	Subdirectory containing LISTEN/NOTIFY status data	|
|	pg_repslot/	|	Subdirectory containing replication slot data. Version 9.4 or later.	|
|	pg_serial/	|	Subdirectory containing information about committed serializable transactions (version 9.1 or later)	|
|	pg_snapshots/	|	Subdirectory containing exported snapshots (version 9.2 or later). The PostgreSQL's function pg_export_snapshot creates a snapshot information file in this subdirectory.	|
|	pg_stat/	|	Subdirectory containing permanent files for the statistics subsystem.	|
|	pg_stat_tmp/	|	Subdirectory containing temporary files for the statistics subsystem.	|
|	pg_subtrans/	|	Subdirectory containing subtransaction status data	|
|	pg_tblspc/	|	Subdirectory containing symbolic links to tablespaces	|
|	pg_twophase/	|	Subdirectory containing state files for prepared transactions	|
|	pg_wal/ (Version 10 or later)	|	Subdirectory containing WAL (Write Ahead Logging) segment files. It is renamed from pg_xlog in Version 10.	|
|	pg_xact/ (Version 10 or later)	|	Subdirectory containing transaction commit state data. It is renamed from pg_clog in Version 10. 	|
|	pg_xlog/ (Version 9.6 or earlier)	|	Subdirectory containing WAL (Write Ahead Logging) segment files. It is renamed to pg_wal in Version 10.	|

### Layout of Databases
一个数据库是在base subdirectory下的子目录；并且数据库目录名称与对应的OIDs一样。例如，上面我们查询的wz数据库的OID是16384，它的子目录名称也是16384。

```
[root@ecs-s6-large-2-linux-20191218010705 base]# pwd
/data/postgres/data/base
[root@ecs-s6-large-2-linux-20191218010705 base]# ll
total 48
drwx------ 2 postgres postgres 8192 Mar  9 11:24 1
drwx------ 2 postgres postgres 8192 Mar  9 11:24 13213
drwx------ 2 postgres postgres 8192 Mar  9 15:11 13214
drwx------ 2 postgres postgres 8192 Mar 16 14:51 16384
```

### Layout of Files Associated with Tables and Indexes

在数据库目录下，每个小于1GB的表或索引是一个单独file。表和索引是由单独的OIDs管理的，这些数据文件是由变量，相对文件节点管理的。表和索引的相对文件节点值并不总是和OIDs匹配。

让我们先看一下test表的OID和relfilenode。

```
wz=# SELECT relname, oid, relfilenode FROM pg_class WHERE relname = 'test';
 relname |  oid  | relfilenode 
---------+-------+-------------
 test    | 16385 |       16385
(1 row)

[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ cd /data/postgres/data/
[postgres@ecs-s6-large-2-linux-20191218010705 data]$ ls -al base/16384/16385 
-rw------- 1 postgres postgres 0 Mar 16 14:51 base/16384/16385

```

表和索引的relfilenode值会因为一些命令（例如，TRUNCATE, REINDEX, CLUSTER）而改变。例如，如果我们truncate表test，PostgreSQL会分配一个新的relfilenode给这个表，移除老数据文件（16385），并且创建新文件（16388）。
```
postgres=# \c wz
You are now connected to database "wz" as user "postgres".
wz=# \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | test | table | postgres
(1 row)

wz=# select relname, oid, relfilenode from pg_class where relname = 'test';
 relname |  oid  | relfilenode 
---------+-------+-------------
 test    | 16385 |       16385
(1 row)

wz=# truncate test;
TRUNCATE TABLE
wz=# select relname, oid, relfilenode from pg_class where relname = 'test';
 relname |  oid  | relfilenode 
---------+-------+-------------
 test    | 16385 |       16388
(1 row)

```

9.0以上版本，内建的pg_relation_filepath可以用来查询表的OID，如下所示。
```
postgres=# \c wz
You are now connected to database "wz" as user "postgres".
wz=# SELECT pg_relation_filepath('test');
 pg_relation_filepath 
----------------------
 base/16384/16388
(1 row)
```

当表和索引的大小超过1GB时，PostgreSQL创建并使用一个类似relfilenode.1名称的新文件。如果这个新文件被填满了，下一个类似relfilenode.2的新文件将会被创建，以此类推。
```
$ cd $PGDATA
$ ls -la -h base/16384/19427*
-rw------- 1 postgres postgres 1.0G  Apr  21 11:16 data/base/16384/19427
-rw------- 1 postgres postgres  45M  Apr  21 11:20 data/base/16384/19427.1
...
```

表和索引的最大文件小大可以通过在编译PostgreSQL软件时指定--with-segsize参数修改。


在数据库子目录中，你可能可以找到每个表有两个后缀为'_fsm'和'_vm'的关联文件。这两个文件对应free space map和visibility map，分别存储每个表文件上的free space capacity和visibility信息。索引只有独立的free space map，没有visibility map。
```
$ cd $PGDATA
$ ls -la base/16384/18751*
-rw------- 1 postgres postgres  8192 Apr 21 10:21 base/16384/18751
-rw------- 1 postgres postgres 24576 Apr 21 10:18 base/16384/18751_fsm
-rw------- 1 postgres postgres  8192 Apr 21 10:18 base/16384/18751_vm
```

### Tablespaces
PostgreSQL中的表空间是在base directory之外的附加数据区域。这个功能在8.0版本开始生效。

![image](./img/2020-03-16-postgresql-structure/structure_3.png)

待续……

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
