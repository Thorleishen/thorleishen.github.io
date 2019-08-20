---
layout: post
title: 'arachni指纹识别（七）'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

由于前几天研究指纹对插件的影响，现在目前所作的工作是将漏扫插件中self.info中的platforms信息填写完整



## 如何查找漏扫插件指纹

1. 通过awvs中xml信息可以查看OS和server指纹
2. 通过注入的内容可以查看OS、DB、server、Langeruage和framework

![皮皮狗](https://raw.githubusercontent.com/Thorleishen/thorleishen.github.io/master/images/2019-08-20-img1.jpg)

## 整理插件

通过填写漏扫插件中self.info中的信息，同时也发现我们项目中有些xml文件缺失或者引用错误



## 总结

认真做事总会有收获，并且我们应该在领导安排的时间计划表提前做好我们本分的事情是最好的，尽量不要超出预期