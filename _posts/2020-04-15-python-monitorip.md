---
layout: post
title: Telegraf+InfluxDB+Grafana+Python实现Oracle实时监控——之监控数据库服务器IP地址存活
category: [python]
tags: [InfluxDB]
---

##### 监控数据库服务器IP地址存活
使用Telegraf+InfluxDB+Grafana+Python的方式监控数据库服务器IP地址存活。

###### 监控程序的目录结构如下
----
主要是在telegraf采集程序中创建congfig、shell和log目录，三个目录的作用如下：
config：存放采集IP地址的telegraf配置文件
shell：存放采集IP地址的python脚本
log：存放采集IP地址的对应log日志

![image](/img/2020-04-15-python-monitorip/monitorip_1.png)

###### python脚本如下
----
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author:wangzhong
 
import platform
import re
import subprocess

def check_platform(ipidx,hostname,ip):
	if (platform.system() == "Windows"):
	  check_alive_windows(ipidx,hostname,ip)
	elif (platform.system() == "Linux"):
		check_alive_linux(ipidx,hostname,ip)
	else:
	  print ("Platform Unsupported")
	  	


def check_alive_windows(ipidx,hostname,ip,count = 4,timeout = 1):
  cmd = 'ping -n %d -w %d %s'%(count,timeout,ip)
  p = subprocess.Popen(cmd,stdin=subprocess.PIPE,stdout=subprocess.PIPE,stderr=subprocess.PIPE,bufsize=1)
  for line in iter(p.stdout.readline, b''):
    result=p.stdout.read()
    regex=re.findall('100% 丢失',result.decode('gbk'))
    
    if len(regex) == 0:
      print ("oracle_srv_state,ipidx=\"%s\",fqdn=\"%s\" dayidx=1,ip=\"%s\",srvstate=\"UP\"" %(ipidx,hostname,ip) ) 
    else:
      print ("oracle_srv_state,ipidx=\"%s\",fqdn=\"%s\" dayidx=1,ip=\"%s\",srvstate=\"DOWN\"" %(ipidx,hostname,ip) ) 
  p.stdout.close()
  p.wait()

def check_alive_linux(ipidx,hostname,ip,count = 4,timeout = 1):
  cmd = 'ping -c %d -w %d %s'%(count,timeout,ip)
  p = subprocess.Popen(cmd,stdin=subprocess.PIPE,stdout=subprocess.PIPE,stderr=subprocess.PIPE,bufsize=1)
  for line in iter(p.stdout.readline, b''):
    result=p.stdout.read()
    regex=re.findall('100% packet loss',result.decode('gbk'))
    
    if len(regex) == 0:
      print ("oracle_srv_state,ipidx=\"%s\",fqdn=\"%s\" dayidx=1,ip=\"%s\",srvstate=\"UP\"" %(ipidx,hostname,ip) ) 
    else:
      print ("oracle_srv_state,ipidx=\"%s\",fqdn=\"%s\" dayidx=1,ip=\"%s\",srvstate=\"DOWN\"" %(ipidx,hostname,ip) ) 
  p.stdout.close()
  p.wait()

if __name__ == "__main__":
   with open('hostip.txt','r') as f:
     for line in f.readlines():
      (ipidx,hostname,ip) = line.strip().split(",")
      check_platform(ipidx,hostname,ip)

```
hostip.txt监控IP清单中的格式如下，以英文逗号“,”作为分隔符，如下所示：
1,testdb01,192.168.1.1

###### Grafana展示页面
----
![image](/img/2020-04-15-python-monitorip/monitorip_2.png)

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
