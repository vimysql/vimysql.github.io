---
layout: post
title: Telegraf+InfluxDB+Grafana+Python实现Oracle实时监控——之InfluxDB
category: [python]
tags: [InfluxDB]
---
##### InfluxDB

###### What is InfluxDB?
----
InfluxDB是一个开源分布式时序、事件和指标数据库。

目前的数据库排名情况如下图所示：
![image](/img/2021-04-06-python-monitorinflux/monitorinflux_1.png)

InfluxDB使用 Go 语言编写，无需外部依赖。其设计目标是实现分布式和水平伸缩扩展。

InfluxDB包括用于存储和查询数据，在后台处理ETL或监视和警报目的，用户仪表板以及可视化和探索数据等的API。

来自官网的介绍如下：

InfluxDB is the essential time series toolkit — dashboards, queries, tasks and agents all in one place.

那我们为什么要使用Influxdb？主要其有如下特征。

如下图所示：
![image](/img/2021-04-06-python-monitorinflux/monitorinflux_2.png)

InfluxDB这些特性也非常适合我们用来作为监控程序的数据存储端。

###### InfluxDB体系
----
InfluxDB的体系如下图所示：
![image](/img/2021-04-06-python-monitorinflux/monitorinflux_3.png)

从上面的体系图中，主要的两个模块是我们需要理解的。
1. InfluxDB：时序数据库，存放监控事件及查询相关数据。
2. Telegraf：采集事件、日志等的代理程序。

###### InfluxDB初步使用
----
我们以Linux平台为例介绍。

influxdb启动命令如下：

```
service influxdb start
```

连接influxdb数据库并查看数据库状态等信息。

使用“-host”参数可以指定希望连接的influxdb服务IP，如果是本地服务，可以直接使用influx命令。如下所示：

```
[root@NJ-DBMGR-02-01 ~]# influx -host 192.168.1.1
Connected to http://192.168.1.1:8086 version 1.8.1
InfluxDB shell version: 1.8.1
> show databases;
name: databases
name
----
_internal
dbmgr
> exit
[root@NJ-DBMGR-02-01 ~]# influx
Connected to http://localhost:8086 version 1.8.1
InfluxDB shell version: 1.8.1
> show databases;
name: databases
name
----
_internal
dbmgr
```

创建、删除及使用数据库，命令如下：
```
create database yourdbname

drop database yourdbname

use yourdbname
```

查询表（influxdb中表定义为measurements）：
```
> use dbmgr
Using database dbmgr
> show measurements
name: measurements
name
----
cpu
disk
diskio
kernel
mem
oracle_rman_state
oracle_srv_state
oracle_tbs_state
oracle_tns_state
processes
swap
system
> 
```

查看、创建、修改和删除数据保留策略：
```
> show retention policies on dbmgr
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 720h0m0s 168h0m0s           1        true
> show retention policies on dw
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 0s       168h0m0s           1        true
> alter retention policy "autogen" on "dw" duration 720h
> show retention policies on dw
name    duration shardGroupDuration replicaN default
----    -------- ------------------ -------- -------
autogen 720h0m0s 168h0m0s           1        true
> drop retention policy autogen on dw
```

###### InfluxDB数据呈现
----
一般情况下，我们通常会选择grafana展示influxdb中的数据。除此外，我们也可以将数据格式化为csv等格式，并通过Oracle数据库外部表的方式去访问。

大致思路是，使用influx脚本生成csv文件，然后通过oracle的外部表去访问数据。

```
[oracle@NJ-DBMGR-02-01 ~]$ more /data/oradata/extshell/oracletnsstate.sh
influx -database dbmgr -execute "select time,collector,ip,port,tnsstate from oracle_tns_state where tnsstate='DOWN'  tz('Asia/Shanghai')"  -precision 'rfc3339'  -format csv  >/data/oradata/exttab/oracletnsstate.csv

[oracle@NJ-DBMGR-02-01 ~]$ influx -database dbmgr -execute "select time,collector,ip,port,tnsstate from oracle_tns_state where tnsstate='DOWN'  tz('Asia/Shanghai')"  -precision 'rfc3339'| more 
name: oracle_tns_state
time                      collector ip          port tnsstate
----                      --------- --          ---- --------
2021-03-07T04:47:01+08:00 telegraf  192.168.1.200 1521 DOWN
2021-03-07T04:47:16+08:00 telegraf  192.168.1.200 1521 DOWN
2021-03-07T04:48:01+08:00 telegraf  192.168.1.200 1521 DOWN
2021-03-07T04:49:01+08:00 telegraf  192.168.1.200 1521 DOWN
2021-03-07T04:50:01+08:00 telegraf  192.168.1.200 1521 DOWN

```
oracle数据库中创建外部表的语句如下：

```
-- Create table
create table T_ORACLETNSSTATE
(
  name         VARCHAR2(30),
  tnstime      VARCHAR2(30),
  tnscollector VARCHAR2(20),
  tnsip        VARCHAR2(20),
  tnsport      VARCHAR2(10),
  tnsstate     VARCHAR2(10)
)
organization external
(
  type ORACLE_LOADER
  default directory EXTTAB
  access parameters 
  (
    fields terminated by ','
  )
  location (EXTTAB:'oracletnsstate.csv')
)
reject limit UNLIMITED;
```


> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
