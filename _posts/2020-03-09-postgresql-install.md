---
layout: post
title: 'PostgreSQL安装'
date: '2020-03-09'
header-img: "img/post-bg-android.jpg"
tags:
     - postgresql
author: '王小胖'
---

# CentOS 7平台安装PostgreSQL 10.12
## 解压软件tar包
----
```
[root@ecs-s6-large-2-linux-20191218010705 software]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
[root@ecs-s6-large-2-linux-20191218010705 software]# ll
total 24360
-rw-rw-r-- 1 root root 24940708 Mar  6 16:55 postgresql-10.12.tar.gz
[root@ecs-s6-large-2-linux-20191218010705 software]# tar -zxvf postgresql-10.12.tar.gz
postgresql-10.12/
postgresql-10.12/.dir-locals.el
postgresql-10.12/contrib/
postgresql-10.12/contrib/tcn/
postgresql-10.12/contrib/tcn/tcn.control
postgresql-10.12/contrib/tcn/Makefile
postgresql-10.12/contrib/tcn/tcn.c
postgresql-10.12/contrib/tcn/tcn--1.0.sql
postgresql-10.12/contrib/sslinfo/ 
```


## 解压后将目录复制到/usr/local目录
----
复制完成后使用ln -s做软链接，方便今后的postgresql软件升级
```[root@ecs-s6-large-2-linux-20191218010705 software]# ll
total 24364
drwxrwxrwx 6 1107 1107     4096 Feb 11 06:32 postgresql-10.12
-rw-rw-r-- 1 root root 24940708 Mar  6 16:55 postgresql-10.12.tar.gz
[root@ecs-s6-large-2-linux-20191218010705 software]# cp -R postgresql-10.12 /usr/local/
[root@ecs-s6-large-2-linux-20191218010705 software]# cd /usr/local/
[root@ecs-s6-large-2-linux-20191218010705 local]# ln -s /usr/local/postgresql-10.12 postgresql
```

## 编译安装postgresql时可能会出现如下两个问题
----
```[root@ecs-s6-large-2-linux-20191218010705 local]# cd postgresql
[root@ecs-s6-large-2-linux-20191218010705 postgresql]#  ./configure   --prefix=/usr/local/postgresql --with-python --with-perl
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking which template to use... linux
checking whether NLS is wanted... no```

出现如下问题，执行yum install perl-ExtUtils-Embed
```checking for flags to link embedded Perl... Can't locate ExtUtils/Embed.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .).
BEGIN failed--compilation aborted.
no
configure: error: could not determine flags for linking embedded Perl.
This probably means that ExtUtils::Embed or ExtUtils::MakeMaker is not
installed.```

出现如下问题，执行yum install python python-devel  
```checking Python.h usability... no
checking Python.h presence... no
checking for Python.h... no
configure: error: header file <Python.h> is required for Python```

问题解决后重新执行./configure   --prefix=/usr/local/postgresql --with-python --with-perl命令即可。
```[root@ecs-s6-large-2-linux-20191218010705 postgresql]#  ./configure   --prefix=/usr/local/postgresql --with-python --with-perl
[root@ecs-s6-large-2-linux-20191218010705 postgresql]# make && make   install```
## 创建postgres用户及用户组，并initdb
----
```[root@ecs-s6-large-2-linux-20191218010705 postgresql]# groupadd postgres
[root@ecs-s6-large-2-linux-20191218010705 postgresql]# useradd -g postgres   postgres
[root@ecs-s6-large-2-linux-20191218010705 postgresql]# id postgres
uid=1000(postgres) gid=1001(postgres) groups=1001(postgres)
[root@ecs-s6-large-2-linux-20191218010705 postgresql]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  8.6M  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/vda1        40G  3.5G   34G  10% /
/dev/vdb1       100G   46M  100G   1% /data
tmpfs           379M     0  379M   0% /run/user/0
[root@ecs-s6-large-2-linux-20191218010705 postgresql]# mkdir -p /data/postgres
[root@ecs-s6-large-2-linux-20191218010705 postgresql]# chown postgres:postgres  /data/postgres/
[root@ecs-s6-large-2-linux-20191218010705 postgresql]# vim /etc/profile
export PATH=/usr/local/postgresql/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/postgresql/lib

[root@ecs-s6-large-2-linux-20191218010705 postgresql]# su - postgres
[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ which initdb
/usr/local/postgresql/bin/initdb
[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ export   PGDATA=/data/postgres/data
[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ initdb
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /data/postgres/data ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default timezone ... Asia/Shanghai
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

pg_ctl -D /data/postgres/data -l logfile start```
    
## 安装contrib目录下的工具
----
```[root@ecs-s6-large-2-linux-20191218010705 postgresql]# cd /usr/local/postgresql/contrib/
[root@ecs-s6-large-2-linux-20191218010705 contrib]# make&&make   install```

## 启动postgresql数据库
----
```[root@ecs-s6-large-2-linux-20191218010705 contrib]# su  - postgres
Last login: Mon Mar  9 11:23:48 CST 2020 on pts/0
[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ pg_ctl -D /data/postgres/data -l logfile start
waiting for server to start.... done
server started
[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ ps -ef | grep postgre
root     19269  3995  0 11:26 pts/0    00:00:00 su - postgres
postgres 19271 19269  0 11:26 pts/0    00:00:00 -bash
postgres 19378     1  0 11:26 pts/0    00:00:00 /usr/local/postgresql-10.12/bin/postgres -D /data/postgres/data
postgres 19380 19378  0 11:26 ?        00:00:00 postgres: checkpointer process   
postgres 19381 19378  0 11:26 ?        00:00:00 postgres: writer process   
postgres 19382 19378  0 11:26 ?        00:00:00 postgres: wal writer process   
postgres 19383 19378  0 11:26 ?        00:00:00 postgres: autovacuum launcher process   
postgres 19384 19378  0 11:26 ?        00:00:00 postgres: stats collector process   
postgres 19385 19378  0 11:26 ?        00:00:00 postgres: bgworker: logical replication launcher   
postgres 19392 19271  0 11:26 pts/0    00:00:00 ps -ef
postgres 19393 19271  0 11:26 pts/0    00:00:00 grep --color=auto postgre
[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ psql
psql (10.12)
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)

postgres=# ```

## postgresql配置允许远程连接
----
```[postgres@ecs-s6-large-2-linux-20191218010705 ~]$ vi /data/postgres/data/postgresql.conf
listen_addresses = '127.0.0.1'              # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
port = 5432                             # (change requires restart)
max_connections = 100                   # (change requires restart)

[postgres@ecs-s6-large-2-linux-20191218010705 data]$ vi /data/postgres/data/pg_hba.conf

[postgres@ecs-s6-large-2-linux-20191218010705 data]$ pg_ctl -D /data/postgres/data restart
waiting for server to shut down....2020-03-09 15:10:06.167 CST [31040] LOG:  received fast shutdown request
2020-03-09 15:10:06.168 CST [31040] LOG:  aborting any active transactions
2020-03-09 15:10:06.169 CST [31040] LOG:  worker process: logical replication launcher (PID 31047) exited with exit code 1
2020-03-09 15:10:06.169 CST [31042] LOG:  shutting down
2020-03-09 15:10:06.184 CST [31040] LOG:  database system is shut down
 done
server stopped
waiting for server to start....2020-03-09 15:10:06.273 CST [31274] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2020-03-09 15:10:06.278 CST [31274] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2020-03-09 15:10:06.288 CST [31275] LOG:  database system was shut down at 2020-03-09 15:10:06 CST
2020-03-09 15:10:06.291 CST [31274] LOG:  database system is ready to accept connections
 done
server started```


> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
