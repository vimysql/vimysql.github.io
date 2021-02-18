---
layout: post
title: Oracle数据库中汉字生僻字如何处理
category: [oracle]
tags: [生僻字]
---

#### 数据库中汉字生僻字
介绍在Oracle如何处理需要录入汉字生僻字的方法

##### 生僻字案例
----
如汉字中的生僻字"𡚸"，在录入数据库中，如果不做特殊处理时，最后会显示为"??"。
如下图所示：
![image](./img/2020-05-15-oracle-shengpizi/shengpizi_1.png)

##### 使用nvarchar字符类型存储
----
```
select '𡚸', n'𡚸', userenv('language') from dual;

```
我们如果想正常保存或者插入生僻字，可以使用n'生僻字'的方式进行操作。
如下图所示：
![image](./img/2020-05-15-oracle-shengpizi/shengpizi_2.png)

> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
