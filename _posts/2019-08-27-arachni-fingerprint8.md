---
layout: post
title: 'arachni指纹识别（八）'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

这几天依然是完善漏扫插件中self.info中的信息，主要讲讲自己遇到的坑



## 踩坑（一）

本以为只需要在self.info中添加platforms字段就可以，但是发现当我们检测到漏洞时会报错，NoMethod "merge"，报错的原因是找不到self.info中issue字段；当检测到漏洞时，会调用log方法，log方法中会调用log_issues方法，再调用create_issues方法，会获取self.info中的issues字段，由于漏扫插件继承Check::Base，如果没有对self.info重写，会继承Check::Base中的self.info,由于我们重写的self.info只有platforms字段，故self.info中没有issues，解决办法，根据Check::Base方法中self.info重写漏扫插件self.info，加上platforms字段：

```ruby
def self.info
	name: %q{},
    description: %q{},
    platforms: [],
    issues:{
        name: %q{},
        description: %q{},
        cwe: 0,
        severy: MEDUME::HIGH
        }

end
```



![皮皮猪](https://raw.githubusercontent.com/Thorleishen/thorleishen.github.io/master/images/2019-08-27-img1.jpg)



## 踩坑（二）

由于之前插件的基本信息存放在xml中，但是在我完善漏扫插件时，发现有些xml对应的漏扫插件有错，导致有些插件没有执行，需要完善xml信息并检查xml与漏扫插件是否是对应的。



## 每日一悟

已经做ruby四个月，不知道还能坚持多久，为了生活！！！🐷