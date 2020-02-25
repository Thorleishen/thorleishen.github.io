---
layout: post
title: 'mysql和sqlite批量插入多条数据'
categories: mysql
tags: mysql sqlite linux work
author: thorleishen
---

* content
{:toc}
## 背景

由于项目在启动时，需要往数据库mysql和sqlite中同步插入数据，而产生非常耗时的操作，经过分析，我们插入数据使用单条插入或者mysql性能本身比较差，并且在该系统上插入数据速度慢



## 思考

1. 通过使用多线程批量插入数据
2. 使用批量插入方法executemany()
3. 通过os.system()方案直接通过终端单条插入数据



## 方案一：通过多线程批量插入数据

我们通过队列获取所有的insert语句，开启多个线程执行sql语句，遇到的坑，报错：由于我们每执行一条语句就会commit，故会出现OptionError报错



## 方案二：使用批量插入方法executemany()

executemany(sql, iteable)，由于有些数据不完整，导致会出现缺失数据，即sql中需要的参数和iteable中元组或列表的数据个数不统一



## 方案三：通过os.system()方案直接通过终端插入数据（待检测）

如果插入数据依旧很慢，那么可能时mysql本身性能慢



## 总结

在思考问题时，应找到问题的原因，对症下药，先分析原因后去思考，后去查阅资料

