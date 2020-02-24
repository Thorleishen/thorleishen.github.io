---
layout: post
title: 'mysql数据库备份'
categories: mysql
tags: mysql linux
author: thorleishen
---

* content
{:toc}
## 前言

数据库备份和数据迁移



## 学习路线

数据库备份

热备份和冷备份：

1. 热备份：在不影响mysql正常运行的情况下进行备份，可能会导致数据库增加操作数据丢失；如mysqldump命令
2. 冷备份：MySQL服务需要关闭，备份效率高；

全量备份和增量备份：

1. 全量备份：将整个数据库迁移过去，但是需要另一台服务的MySQL配置需要保持和当前MySQL服务配置一样；mysqldump命令
2. 增量备份：通过binlog日志，将后续增删操作进行记录日志，备份速度快



## 数据迁移
由于需要从别人服务器备份数据，在不影响别人正常业务运行情况下，只能选择热备份，并且由于权限问题，拿不到binlog日志，不能增量备份；因此，只能通过全量备份进行数据迁移，并通过ssh远程传输备份文件



### 问题一

问题：当前用户的存储空间有限，导致直接全量备份服务器容量不足

解决方法：通过压缩备份的方式进行备份

```shell
-- 数据备份
mysqldump -uroot -p dbname table_name1 | gzip > name.sql.gz
# 不止支持gzip压缩格式，其它主流的也支持，如zip
-- 数据还原
gunzip < name.sql.gz | mysql -uroot -p dbname
```



### 问题二

问题：通过scp命令ssh远程上传文件需要输入密码问题

解决办法：预研发现，linux下expect命令可以进行终端交互操作，在shell脚本中嵌套使用可以做到自动输入密码问题

```shell
# expect命令
# eg:
/usr/bin/expect << EOF
set timeout 30  # timeout指每一条命令执行时间，超过该时间则立即结束该命令并执行下一条命令，-1标识无限大
# spawn表示执行的交互命令
spawn ssh -p18330 root@192.168.10.22
# expect指终端打印的信息
expect {
"*yes/no" { send "yes\r"; exp_continue }
"*password:" { send "$passwd\r" }
}
expect "*#"
# send向终端输入的信息
send "cd /home/trunk\r"
expect "*#"
send "svn up\r"
expect "*#"
send "exit\r"
# interact 使用户处于进程或程序的交互状态，ssh登录后不自动登出
interact
# 退出
expect eof
EOF
```



### 问题三

问题：解决xshell关闭终端窗口，程序被杀死的问题

解决方法：通过nohup让程序在后台执行，或者使用screen命令挂起程序

```shell
# nohup挂起程序
nohup 操作 & # 默认输出的内容存放在nohup.out文件中
# screen挂起程序
screen -ls # 查看所有screen程序
screen -S name # 创建指定名字的screen程序
screen -r name # 进入screen挂起的程序
screen -d name # 脱离screen程序
screen -X -S 122128 quit # 删除一个screen程序
```



## 总结

通过实际情况选择较优的数据迁移方法，在通过脚本实现自动化的数据备份