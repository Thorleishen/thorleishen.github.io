---
layout: post
title: 'arachni指纹识别优化（一）'
categories:arachni
tags: ruby arachni linux
author: thorleishen
---

* content
{:toc}


## 前言

arachni指纹识别的流程及web指纹识别的背景

## fingerprint



- 指纹识别深入细节及原理背景
  - 背景
    - 大多Web攻击主要利用Web服务器及引用存在的安全弱点，进一步利用响应的攻击手法获取目标站点的高级权限和敏感数据。
  - 定义
    - Web指纹是Web服务组件再开发时留下的可对其类型及版本进行标识的特殊信息，包括Web服务器指纹、Web应用指纹以及前端框架指纹等；Web指纹识别是指利用已有的Web指纹库实现对Web服务组件相关信息的准确识别。
  - 作用
    - Web指纹识别技术可用于Web渗透的信息收集环节，准确获取Web服务组件的类型及版本信息可有效检测其存在的安全弱点。
  - 资源来源：<http://www.doc88.com/p-5045696259945.html>



- arachni原始指纹识别插件
  - 插件中options中会有关于指纹识别的相关信息，在auditor.rb中self.log函数中



- 了解其他指纹识别插件并进行对比，w3af、blindElephant等
  - blindElephant
    - BlindElephant web应用程序指纹识别器通过将已知位置的静态文件与所有可用版本中这些文件的版本的预计算哈希值进行比较，从而发现（已知）Web应用程序的版本。该技术快速、低宽带、非侵入性、通用且高度自动化。
  - w3af
    - 过程
      1. 识别所有的链接、表单、查询字符串参数
      2. 将特制字符串发送到每个输入并分析输出
      3. 生成包含调查结果的报告
