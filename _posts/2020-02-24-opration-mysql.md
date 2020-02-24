---
layout: post
title: 'mysql查看数据库大小'
categories: mysql
tags: mysql linux
author: thorleishen
---

* content
{:toc}
## 学习路线

最近在公司进行数据迁移，由于数据维度比较多，导致联表查询和查看数据库大小成了必要的知识问题



## MySQL查询数据库大小和常见问题

1. mysql查询数据库名和查询数据库对应的表名

   ```mysql
   -- 查询某数据库中所有表
   select count(*) from information_schema.TABLES t where t.TABLE_SCHEMA ="数据库名" and t.TABLE_NAME ="数据库表名";
   -- 模糊查询某数据库中的表
   select count(*) from information_schema.TABLES t where t.TABLE_SCHEMA ="数据库名" and t.TABLE_NAME like "%数据库表名%";
   ```

2. mysql聚合查询报错

   ```mysql
   select count(*) from (select * from dede_spacemoney group by sid) ;
   -- 当我执行到这里的时候就抛出了这个异常，原来我进行嵌套查询的时候子查询出来的的结果是作为一个派生表来进行上一级的查询的，所以子查询的结果必须要有一个别名
   
   -- 把MySQL语句改成：
   select count(*) from (select * from list where name="xiao") as t;
   
   -- 问题就解决了，虽然只加了一个没有任何作用的别名t，但这个别名是必须的！
   ```

3. 查询数据库表结构大小

   ```mysqL
   -- 1、进入information_schema 数据库（存放了其他的数据库的信息）
   
   use information_schema;
   
      
   
   -- 2、查询所有数据的大小：
   
   select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables;
   
      
   
   -- 3、查看指定数据库的大小：比如查看数据库home的大小
   
   select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home';
   
      
   
   -- 4、查看指定数据库的某个表的大小，比如查看数据库home中 members 表的大小
   
   select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home' and table_name='members';
   ```

   数据库5.7安装：https://www.cnblogs.com/fanshudada/p/9781794.html



## 总结

由于自身只对简单的增删改查熟悉，遇到其它查找操作，就短路了，后续要进一步学习