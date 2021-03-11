---
layout: post
title: GoldenGate抽取进程无法自动清理trail file问题分析
category: [oracle]
tags: [goldengate]
---
##### GoldenGate purgeoldextracts

###### 设置了purgeoldextracts参数不生效？
----
配置了purgeoldextracts参数后，goldengate抽取进程没有正常清理过旧的trail file。

###### 查询抽取进程相关信息
----
Trail Name信息中竟然有两行信息，并且第一行Seqno和RBA都是0，这个trail file是有问题的。

```
GGSCI (NJ-BDDJDB-01-01) 3> info ext_bi,detail

EXTRACT    EXT_BI    Last Started 2020-12-29 19:30   Status RUNNING
Checkpoint Lag       00:00:00 (updated 00:00:00 ago)
Process ID           448717
Log Read Checkpoint  Oracle Redo Logs
                     2021-03-11 10:12:42  Seqno 7404, RBA 215451648
                     SCN 3123.2310539519 (13415493404927)

  Target Extract Trails:

  Trail Name                                       Seqno        RBA     Max MB Trail Type

  /u01/app/ogg/dirdatmis/bi                            0          0        500 EXTTRAIL  
  ./dirdatmis/bi                                    1102   50268220        500 EXTTRAIL  

  Extract Source                          Begin             End             

  /oradata/bddbogg/bddbogg/redo04.log     2020-12-29 19:29  2021-03-11 10:12
  /oradata/bddbogg/bddbogg/redo01.log     * Initialized *   2020-12-29 19:29
  /oradata/bddbogg/bddbogg/redo01.log     * Initialized *   2020-12-29 19:29
  Not Available                           * Initialized *   2020-12-29 19:29
  Not Available                           * Initialized *   2020-12-29 19:29
  Not Available                           * Initialized *   2020-12-29 19:29

```

确认一下，查询下对应的checkpoint信息。

可以看到分别有Write Checkpoint #1和Write Checkpoint #2，对应不同的Extract Trail。

```
GGSCI (NJ-BDDJDB-01-01) 2> info ext_bi,showch

EXTRACT    EXT_BI    Last Started 2020-12-29 19:30   Status RUNNING
Checkpoint Lag       00:00:00 (updated 00:00:08 ago)
Process ID           448717
Log Read Checkpoint  Oracle Redo Logs
                     2021-03-11 10:30:18  Seqno 7404, RBA 367094272
                     SCN 3123.2405458824 (13415588324232)


Current Checkpoint Detail:

Read Checkpoint #1

  Oracle Redo Log

  Startup Checkpoint (starting position in the data source):
    Thread #: 1
    Sequence #: 5405
    RBA: 597970960
    Timestamp: 2020-12-29 19:29:47.000000
    SCN: Not available
    Redo File: /oradata/bddbogg/bddbogg/redo01.log

  Recovery Checkpoint (position of oldest unprocessed transaction in the data source):
    Thread #: 1
    Sequence #: 7404
    RBA: 366579216
    Timestamp: 2021-03-11 10:30:18.000000
    SCN: 3123.2405458739 (13415588324147)
    Redo File: /oradata/bddbogg/bddbogg/redo04.log

  Current Checkpoint (position of last record read in the data source):
    Thread #: 1
    Sequence #: 7404
    RBA: 367094272
    Timestamp: 2021-03-11 10:30:18.000000
    SCN: 3123.2405458824 (13415588324232)
    Redo File: /oradata/bddbogg/bddbogg/redo04.log

  BR Previous Recovery Checkpoint:
    Thread #: 0
    Sequence #: 0
    RBA: 0
    Timestamp: 2020-12-29 19:30:47.707485
    SCN: Not available
    Redo File: 

  BR Begin Recovery Checkpoint:
    Thread #: 1
    Sequence #: 7401
    RBA: 393718784
    Timestamp: 2021-03-11 08:05:05.000000
    SCN: 3123.1952623840 (13415135489248)
    Redo File: 

  BR End Recovery Checkpoint:
    Thread #: 1
    Sequence #: 7401
    RBA: 393718784
    Timestamp: 2021-03-11 08:05:05.000000
    SCN: 3123.1952623840 (13415135489248)
    Redo File: 

Write Checkpoint #1

  GGS Log Trail

  Current Checkpoint (current write position):
    Sequence #: 0
    RBA: 0
    Timestamp: 2020-12-29 19:29:54.315268
    Extract Trail: /u01/app/ogg/dirdatmis/bi
    Seqno Length: 6
    Flip Seqno Length: Yes
    Trail Type: EXTTRAIL

Write Checkpoint #2

  GGS Log Trail

  Current Checkpoint (current write position):
    Sequence #: 1102
    RBA: 98001999
    Timestamp: 2021-03-11 10:30:21.230018
    Extract Trail: ./dirdatmis/bi
    Seqno Length: 9
    Flip Seqno Length: No
    Trail Type: EXTTRAIL

Header:
  Version = 2
  Record Source = A
  Type = 10
  # Input Checkpoints = 1
  # Output Checkpoints = 2

File Information:
  Block Size = 2048
  Max Blocks = 100
  Record Length = 2048
  Current Offset = 0

Configuration:
  Data Source = 3
  Transaction Integrity = 1
  Task Type = 0

Status:
  Start Time = 2020-12-29 19:30:47
  Last Update Time = 2021-03-11 10:30:21
  Stop Status = A
  Last Result = 400
```
###### 删除有问题的Extract Trail，删除后过旧的trail file就能按照清理策略删除了。
----
```
GGSCI (NJ-BDDJDB-01-01) 4> stop ext_bi

Sending STOP request to EXTRACT EXT_BI ...
Request processed.


GGSCI (NJ-BDDJDB-01-01) 5> delete exttrail /u01/app/ogg/dirdatmis/bi
Deleting extract trail /u01/app/ogg/dirdatmis/bi for extract EXT_BI


GGSCI (NJ-BDDJDB-01-01) 7> info ext_bi,showch

EXTRACT    EXT_BI    Last Started 2020-12-29 19:30   Status STOPPED
Checkpoint Lag       00:00:00 (updated 00:00:38 ago)
Log Read Checkpoint  Oracle Redo Logs
                     2021-03-11 10:37:20  Seqno 7404, RBA 446590976
                     SCN 3123.2405469907 (13415588335315)


Current Checkpoint Detail:

Read Checkpoint #1

  Oracle Redo Log

  Startup Checkpoint (starting position in the data source):
    Thread #: 1
    Sequence #: 5405
    RBA: 597970960
    Timestamp: 2020-12-29 19:29:47.000000
    SCN: Not available
    Redo File: /oradata/bddbogg/bddbogg/redo01.log

  Recovery Checkpoint (position of oldest unprocessed transaction in the data source):
    Thread #: 1
    Sequence #: 7404
    RBA: 446579728
    Timestamp: 2021-03-11 10:37:20.000000
    SCN: 3123.2405469906 (13415588335314)
    Redo File: /oradata/bddbogg/bddbogg/redo04.log

  Current Checkpoint (position of last record read in the data source):
    Thread #: 1
    Sequence #: 7404
    RBA: 446590976
    Timestamp: 2021-03-11 10:37:20.000000
    SCN: 3123.2405469907 (13415588335315)
    Redo File: /oradata/bddbogg/bddbogg/redo04.log

  BR Previous Recovery Checkpoint:
    Thread #: 0
    Sequence #: 0
    RBA: 0
    Timestamp: 2020-12-29 19:30:47.707485
    SCN: Not available
    Redo File: 

  BR Begin Recovery Checkpoint:
    Thread #: 1
    Sequence #: 7401
    RBA: 393718784
    Timestamp: 2021-03-11 08:05:05.000000
    SCN: 3123.1952623840 (13415135489248)
    Redo File: 

  BR End Recovery Checkpoint:
    Thread #: 1
    Sequence #: 7401
    RBA: 393718784
    Timestamp: 2021-03-11 08:05:05.000000
    SCN: 3123.1952623840 (13415135489248)
    Redo File: 

Write Checkpoint #1

  GGS Log Trail

  Current Checkpoint (current write position):
    Sequence #: 1102
    RBA: 126365776
    Timestamp: 2021-03-11 10:37:23.457875
    Extract Trail: ./dirdatmis/bi
    Seqno Length: 9
    Flip Seqno Length: No
    Trail Type: EXTTRAIL

Header:
  Version = 2
  Record Source = A
  Type = 10
  # Input Checkpoints = 1
  # Output Checkpoints = 1

File Information:
  Block Size = 2048
  Max Blocks = 100
  Record Length = 2048
  Current Offset = 0

Configuration:
  Data Source = 3
  Transaction Integrity = 1
  Task Type = 0

Status:
  Start Time = 2020-12-29 19:30:47
  Last Update Time = 2021-03-11 10:37:23
  Stop Status = G
  Last Result = 400



GGSCI (NJ-BDDJDB-01-01) 8> start *

Sending START request to MANAGER ...
EXTRACT EXT_BI starting
EXTRACT PMP_BI is already running.
REPLICAT RPL_ALL is already running.
```


> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
