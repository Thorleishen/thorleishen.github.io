---
layout: post
title: 'arachni插件移植的坑'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

来自arachni的坑，由于要移植web漏扫插件到新的机器上，即arachni框架的移植，需要配置环境，经过前两天的基础环境搭建，程序已经可以跑起来，但是还是出现各种bug，有来自scanpoc插件，即python插件，也有依赖包的问题



## 问题一

依赖包的问题，ffi依赖包报错，找不到require 'xxxx'，即c编译的函数

解决办法：将引用该函数的部分注释掉，这个接口主要是调用我们的告警接口，告警漏洞信息的



## 问题二

找不到rjb（1.5.5）依赖包

原因：主要是由于rjb需要java环境，但是由于我们移植的机器没有联网，并且类似java、mysql、sqilte3等环境都没有

解决办法：将引用该依赖包的文件删掉，主要是wez.rb（这里也有一些记不清，具体还是要看文件哪里有引用）文件



## 问题三

scanpoc插件报错

原因：主要是默认要执行scanpoc插件，并且是python写的；scanpoc插件中引用了pymysql模块，机器上没有python相关的模块

解决办法：删除相应的插件，不执行scanpoc上的插件



## 自我认识

web漏扫的有些配置文件的作用不知道，并且底层知识的匮乏导致工作效率低下，不知还能在这个公司呆多久，只希望多学习一点知识！！！

​																																				

​																																				给以后努力工作的自己！！！

