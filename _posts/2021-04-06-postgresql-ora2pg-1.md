---
layout: post
title: 使用Ora2Pg工具把数据从Oracle导入PostgreSQL
category: [postgresql]
tags: [Ora2Pg]
---
##### Ora2Pg

###### Ora2Pg是什么?
----
Ora2Pg是一个开源工具，可以用来迁移Oracle或者MySQL数据库到PostgreSQL。通过它连接到你的Oracle数据库，自动扫描和抽取数据结构，然后生成可以载入PostgreSQL的SQL脚本。

包含的特性如下：

FEATURES INCLUDED
1. Export full database schema (tables, views, sequences, indexes), with unique, primary, foreign key and check constraints.
2. Export grants/privileges for users and groups.
3. Export range/list partitions and sub partitions.
4. Export a table selection (by specifying the table names).
5. Export Oracle schema to a PostgreSQL 8.4+ schema.
6. Export predefined functions, triggers, procedures, packages and package bodies.
7. Export full data or following a WHERE clause.
8. Full support of Oracle BLOB object as PG BYTEA.
9. Export Oracle views as PG tables.
10. Export Oracle user defined types.
11. Provide some basic automatic conversion of PLSQL code to PLPGSQL.
12. Works on any plateform.
13. Export Oracle tables as foreign data wrapper tables.
14. Export materialized view.
15. Show a detailled report of an Oracle database content.
16. Migration cost assessment of an Oracle database.
17. Migration difficulty level assessment of an Oracle database.
18. Migration cost assessment of PL/SQL code from a file.
19. Migration cost assessment of Oracle SQL queries stored in a file.
20. Generate XML ktr files to be used with Penthalo Data Integrator (Kettle)
21. Export Oracle locator and spatial geometries into PostGis.
22. Export DBLINK as Oracle FDW.
23. Export SYNONYMS as views.
24. Export DIRECTORY as external table or directory for external_file extension.
25. Full MySQL export just like Oracle database.

###### 安装环境及相关软件版本
----
安装环境如下：
```
[root@localhost ~]# cat /etc/redhat-release
Red Hat Enterprise Linux Server release 7.6 (Maipo)


[oracle@localhost ~]$ sqlplus -v

SQL*Plus: Release 12.1.0.2.0 Production


[oracle@localhost ~]$ perl -v

This is perl 5, version 16, subversion 3 (v5.16.3) built for x86_64-linux-thread-multi
(with 39 registered patches, see perl -V for more detail)

Copyright 1987-2012, Larry Wall

Perl may be copied only under the terms of either the Artistic License or the
GNU General Public License, which may be found in the Perl 5 source kit.

Complete documentation for Perl, including FAQ lists, should be found on
this system using "man perl" or "perldoc perl".  If you have access to the
Internet, point your browser at http://www.perl.org/, the Perl Home Page.

```

软件版本如下：
DBI-1.643.tar.gz
DBD-Oracle-1.74.tar.gz
ora2pg-21.1.tar.gz

###### 安装DBI、DBD::Oracle和ora2pg
----
安装DBI
```
[root@localhost ora2pg]# tar -zxvf DBI-1.643.tar.gz
...

[root@localhost ora2pg]# cd DBI-1.643
[root@localhost DBI-1.643]# perl Makefile.PL
[root@localhost DBI-1.643]# make
[root@localhost DBI-1.643]# make test
[root@localhost DBI-1.643]# make install

```

安装DBD::Oracle
配置环境变量
```
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/12c/db
export LD_LIBRARY_PATH=/u01/app/oracle/product/12c/db/lib
```

安装DBD::Oracle
```
[root@localhost ora2pg]# tar -zxvf DBD-Oracle-1.74.tar.gz
...

[root@localhost ora2pg]# cd DBD-Oracle-1.74
[root@localhost DBD-Oracle-1.74]# perl Makefile.PL
[root@localhost DBD-Oracle-1.74]# make
[root@localhost DBD-Oracle-1.74]# make test
[root@localhost DBD-Oracle-1.74]# make install

```

安装ora2pg
```
[root@localhost ora2pg]# tar -zxvf ora2pg-21.1.tar.gz
...

[root@localhost ora2pg]# cd ora2pg-21.1
[root@localhost ora2pg-21.1]# perl Makefile.PL
[root@localhost ora2pg-21.1]# make && make install
```

安装后使用脚本list.pl检查
```
[root@localhost software]# cat list.pl
#!/usr/bin/perl
use strict;
use ExtUtils::Installed;
my $inst= ExtUtils::Installed->new();
my @modules = $inst->modules();
foreach(@modules)
{
        my $ver = $inst->version($_) || "???";
        printf("%-12s --  %s\n", $_, $ver);  
}
exit;

[root@localhost software]# perl list.pl
DBD::Oracle  --  1.74
DBI          --  1.643
Ora2Pg       --  21.1
Perl         --  5.16.3
```

###### Ora2Pg导出Oracle数据测试
----
修改ora2pg配置参数
```
[root@localhost software]# cp /etc/ora2pg/ora2pg.conf.dist /home/oracle/ora2pg.conf
[root@localhost software]# cd /home/oracle
[root@localhost software]# vi ora2pg.conf
ORACLE_HOME	    /u01/app/oracle/product/12c/db
ORACLE_DSN	    dbi:Oracle:host=192.168.56.111;sid=testdb
ORACLE_USER	    system
ORACLE_PWD	    oracle123
SCHEMA          wz
USER_GRANTS     0
DEBUG		        0
ORA_INITIAL_COMMAND
EXPORT_SCHEMA	  0 
CREATE_SCHEMA	  1
COMPILE_SCHEMA	0
TYPE		        TABLE,INSERT
OUTPUT		      output.sql
```

参数中可以看出目前支持如下数据类型
```
# EXPORT SECTION (Export type and filters)
#------------------------------------------------------------------------------

# Type of export. Values can be the following keyword:
#       TABLE           Export tables, constraints, indexes, ...
#       PACKAGE         Export packages
#       INSERT          Export data from table as INSERT statement
#       COPY            Export data from table as COPY statement
#       VIEW            Export views
#       GRANT           Export grants
#       SEQUENCE        Export sequences
#       TRIGGER         Export triggers
#       FUNCTION        Export functions
#       PROCEDURE       Export procedures
#       TABLESPACE      Export tablespace (PostgreSQL >= 8 only)
#       TYPE            Export user defined Oracle types
#       PARTITION       Export range or list partition (PostgreSQL >= v8.4)
#       FDW             Export table as foreign data wrapper tables
#       MVIEW           Export materialized view as snapshot refresh view
#       QUERY           Convert Oracle SQL queries from a file.
#       KETTLE          Generate XML ktr template files to be used by Kettle.
#       DBLINK          Generate oracle foreign data wrapper server to use as dblink.
#       SYNONYM         Export Oracle's synonyms as views on other schema's objects.
#       DIRECTORY       Export Oracle's directories as external_file extension objects.
#       LOAD            Dispatch a list of queries over multiple PostgreSQl connections.
#       TEST            perform a diff between Oracle and PostgreSQL database.
#       TEST_VIEW       perform a count on both side of rows returned by views

```

ora2pg导出Oracle表数据
```
[root@localhost oracle]# ora2pg -c ora2pg.conf
[========================>] 1/1 tables (100.0%) end of scanning.
Retrieving table partitioning information...
[>                        ] 0/1 tables (0.0%) end of scanning.
[========================>] 1/1 tables (100.0%) end of table export.
[========================>] 33/33 rows (100.0%) Table TEST (33 recs/sec)
[========================>] 33/33 total rows (100.0%) - (0 sec., avg: 33 recs/sec).
[========================>] 33/33 rows (100.0%) on total estimated data (1 sec., avg: 33 recs/sec)
```

导出后会分别生成表定义和数据INSERT脚本
```
[root@localhost oracle]# ls
INSERT_output.sql  ora2pg.conf  perl5  TABLE_output.sql

[root@localhost oracle]# cat TABLE_output.sql
-- Generated by Ora2Pg, the Oracle database Schema converter, version 21.1
-- Copyright 2000-2020 Gilles DAROLD. All rights reserved.
-- DATASOURCE: dbi:Oracle:host=192.168.56.111;sid=testdb;port=1521

SET client_encoding TO 'UTF8';

\set ON_ERROR_STOP ON


CREATE TABLE test (
        username varchar(128) NOT NULL,
        user_id bigint NOT NULL,
        password varchar(4000),
        account_status varchar(32) NOT NULL,
        lock_date timestamp,
        expiry_date timestamp,
        default_tablespace varchar(30) NOT NULL,
        temporary_tablespace varchar(30) NOT NULL,
        created timestamp NOT NULL,
        profile varchar(128) NOT NULL,
        initial_rsrc_consumer_group varchar(128),
        external_name varchar(4000),
        password_versions varchar(12),
        editions_enabled varchar(1),
        authentication_type varchar(8),
        proxy_only_connect varchar(1),
        common varchar(3),
        last_login timestamp with time zone,
        oracle_maintained varchar(1)
) ;
[root@localhost oracle]# cat INSERT_output.sql

BEGIN;
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'WZ',97,NULL,E'OPEN',NULL,'2021-10-10 16:29:21',E'USERS',E'TEMP','2021-04-13 16:29:21',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'NO',NULL,E'N');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'ORACLE_OCM',36,NULL,E'EXPIRED & LOCKED','2021-03-06 00:10:03','2021-03-06 00:10:03',E'USERS',E'TEMP','2021-03-06 00:10:03',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'OJVMSYS',70,NULL,E'EXPIRED & LOCKED','2021-03-06 00:17:29','2021-03-06 00:17:29',E'USERS',E'TEMP','2021-03-06 00:17:29',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SYSKM',2147483619,NULL,E'EXPIRED & LOCKED','2021-03-06 00:08:36','2021-03-06 00:08:36',E'USERS',E'TEMP','2021-03-06 00:08:36',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'XS$NULL',2147483638,NULL,E'EXPIRED & LOCKED','2021-03-06 00:09:51','2021-03-06 00:09:51',E'USERS',E'TEMP','2021-03-06 00:09:51',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'11G ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'GSMCATUSER',61,NULL,E'EXPIRED & LOCKED','2021-03-06 00:16:38','2021-03-06 00:16:38',E'USERS',E'TEMP','2021-03-06 00:16:38',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'MDDATA',85,NULL,E'EXPIRED & LOCKED','2021-03-06 00:59:36','2021-03-06 00:59:36',E'USERS',E'TEMP','2021-03-06 00:27:34',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SYSBACKUP',2147483617,NULL,E'EXPIRED & LOCKED','2021-03-06 00:08:36','2021-03-06 00:08:36',E'USERS',E'TEMP','2021-03-06 00:08:36',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'DIP',23,NULL,E'EXPIRED & LOCKED','2021-03-06 00:09:25','2021-03-06 00:09:25',E'USERS',E'TEMP','2021-03-06 00:09:25',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SYSDG',2147483618,NULL,E'EXPIRED & LOCKED','2021-03-06 00:08:36','2021-03-06 00:08:36',E'USERS',E'TEMP','2021-03-06 00:08:36',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'APEX_PUBLIC_USER',93,NULL,E'EXPIRED & LOCKED','2021-03-06 00:31:59','2021-03-06 00:31:59',E'USERS',E'TEMP','2021-03-06 00:31:59',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SPATIAL_CSW_ADMIN_USR',90,NULL,E'EXPIRED & LOCKED','2021-03-06 00:31:21','2021-03-06 00:31:21',E'USERS',E'TEMP','2021-03-06 00:31:21',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SPATIAL_WFS_ADMIN_USR',87,NULL,E'EXPIRED & LOCKED','2021-03-06 00:31:15','2021-03-06 00:31:15',E'USERS',E'TEMP','2021-03-06 00:31:15',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'GSMUSER',22,NULL,E'EXPIRED & LOCKED','2021-03-06 00:09:19','2021-03-06 00:09:19',E'USERS',E'TEMP','2021-03-06 00:09:19',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'AUDSYS',7,NULL,E'EXPIRED & LOCKED','2021-03-06 00:08:36','2021-03-06 00:08:36',E'USERS',E'TEMP','2021-03-06 00:08:36',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'FLOWS_FILES',92,NULL,E'EXPIRED & LOCKED','2021-03-06 00:35:00','2021-03-06 00:35:00',E'SYSAUX',E'TEMP','2021-03-06 00:31:59',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'MDSYS',79,NULL,E'EXPIRED & LOCKED','2021-03-06 00:20:35','2021-03-06 00:20:35',E'SYSAUX',E'TEMP','2021-03-06 00:20:35',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'ORDSYS',75,NULL,E'EXPIRED & LOCKED','2021-03-06 00:20:35','2021-03-06 00:20:35',E'SYSAUX',E'TEMP','2021-03-06 00:20:35',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'DBSNMP',48,NULL,E'EXPIRED & LOCKED','2021-03-06 00:13:57','2021-03-06 00:13:57',E'SYSAUX',E'TEMP','2021-03-06 00:13:57',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'WMSYS',62,NULL,E'EXPIRED & LOCKED','2021-03-06 00:16:45','2021-03-06 00:16:45',E'SYSAUX',E'TEMP','2021-03-06 00:16:45',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'APEX_040200',96,NULL,E'EXPIRED & LOCKED','2021-03-06 00:35:00','2021-03-06 00:35:00',E'SYSAUX',E'TEMP','2021-03-06 00:31:59',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'APPQOSSYS',49,NULL,E'EXPIRED & LOCKED','2021-03-06 00:13:58','2021-03-06 00:13:58',E'SYSAUX',E'TEMP','2021-03-06 00:13:58',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'GSMADMIN_INTERNAL',21,NULL,E'EXPIRED & LOCKED','2021-03-06 00:09:19','2021-03-06 00:09:19',E'SYSAUX',E'TEMP','2021-03-06 00:09:19',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'ORDDATA',76,NULL,E'EXPIRED & LOCKED','2021-03-06 00:20:35','2021-03-06 00:20:35',E'SYSAUX',E'TEMP','2021-03-06 00:20:35',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'CTXSYS',73,NULL,E'EXPIRED & LOCKED','2021-03-06 00:20:34','2021-03-06 00:20:34',E'SYSAUX',E'TEMP','2021-03-06 00:19:42',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES','2021-03-06 00:20:32.000000 +08:00',E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'ANONYMOUS',51,NULL,E'EXPIRED & LOCKED','2021-03-06 00:59:36','2021-03-06 00:59:36',E'SYSAUX',E'TEMP','2021-03-06 00:14:02',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'XDB',50,NULL,E'EXPIRED & LOCKED','2021-03-06 00:14:02','2021-03-06 00:14:02',E'SYSAUX',E'TEMP','2021-03-06 00:14:02',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'ORDPLUGINS',77,NULL,E'EXPIRED & LOCKED','2021-03-06 00:20:35','2021-03-06 00:20:35',E'SYSAUX',E'TEMP','2021-03-06 00:20:35',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SI_INFORMTN_SCHEMA',78,NULL,E'EXPIRED & LOCKED','2021-03-06 00:20:35','2021-03-06 00:20:35',E'SYSAUX',E'TEMP','2021-03-06 00:20:35',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'OLAPSYS',82,NULL,E'EXPIRED & LOCKED','2021-03-06 00:27:24','2021-03-06 00:27:24',E'SYSAUX',E'TEMP','2021-03-06 00:27:24',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'OUTLN',13,NULL,E'EXPIRED & LOCKED','2021-03-06 00:08:37','2021-03-06 00:08:37',E'SYSTEM',E'TEMP','2021-03-06 00:08:37',E'DEFAULT',E'DEFAULT_CONSUMER_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SYSTEM',8,NULL,E'OPEN',NULL,'2021-09-02 00:08:36',E'SYSTEM',E'TEMP','2021-03-06 00:08:36',E'DEFAULT',E'SYS_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES','2021-04-13 16:27:26.000000 +08:00',E'Y');
INSERT INTO test (username,user_id,password,account_status,lock_date,expiry_date,default_tablespace,temporary_tablespace,created,profile,initial_rsrc_consumer_group,external_name,password_versions,editions_enabled,authentication_type,proxy_only_connect,common,last_login,oracle_maintained) OVERRIDING SYSTEM VALUE  VALUES (E'SYS',0,NULL,E'OPEN',NULL,'2021-09-02 00:08:36',E'SYSTEM',E'TEMP','2021-03-06 00:08:36',E'DEFAULT',E'SYS_GROUP',NULL,E'10G 11G 12C ',E'N',E'PASSWORD',E'N',E'YES',NULL,E'Y');

COMMIT;
```
###### PostgreSQL导入数据
----
psql导入数据，可以将TABLE_output.sql和INSERT_output.sql脚本合并为output.sql
```
[root@localhost pgsql]# psql wz wz < output.sql
```

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
