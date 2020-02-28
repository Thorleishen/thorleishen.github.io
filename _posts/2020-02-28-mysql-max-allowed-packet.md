---
layout: post
title: 'MySQL导入最大文件设置'
categories: mysql
tags: mysql linux
author: thorleishen
---

* content
{:toc}
## 学习路线

数据备份，导入备份数据，验证数据数据的正确性



## 遇到的问题

在mysql中使用gzip压缩文件导入sql.gz文件时报错ERROR 1153 (08S01): Got a packet bigger than 'max_allowed_packet'



## MySQL最大文件配置配置

修改文件方法

1. 修改/etc/my.cnf文件

   ```ini
   wait_timeout = 2880000
   
   interactive_timeout = 2880000
   
   max_allowed_packet = 128M //1204*1024*128
   ```

2. 直接在数据库中修改

   ```mysql
   --  查看当前大小
   show global variables like 'max_allowed_packet';
   -- 在命令行使用临时设置大小
   set global max_allowed_packet=1024*1024*400;
   ```

数据库重启服务

```shell
service mysql restart
/etc/init.d/mysql restart
systemctl restart mysql # 需要添加mysqld.service服务
```



## 总结

数据库备份不全，可能是因为备份数据库文件过大，或者导入数据超时