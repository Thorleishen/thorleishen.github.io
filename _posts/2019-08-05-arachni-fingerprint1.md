---
layout: post
title: 'arachni指纹识别'
categories: arachni
tags: ruby arachni linux
author: thorleishen
---

* content
{:toc}
## 前言

arachni指定指纹识别插件：需求预演，前端通过配置项指定要检测的指纹识别插件，需要学习arachni指纹识别整体流程



## arachni指纹识别流程

### 1、parse_options处理CLI(command-line-interface，命令行界面)

位置：ui/cli/framework.rb

流程：调用parse_options函数  ->  实例化OptionParser  ->  通过调用OptionParser中各命令行传参  ->  检测是否指定checks插件，没有则初始化所有插件  -> 检测是否指定plugins插件，没有则初始化默认插件（注：checks中默认导入所有插件，而component中默认导入default中的plugins插件）    ->  检测是否指定指纹识别插件



### 2、crawl_url页面的爬取

1. 流程：crawl_url  ->  lib/topsec/framework/parts/audit.rb：crawl_url  ->  pop_page将起始url放入队列中  ->  crawl_page通过请求起始url获取子url  ->  push_paths_from_page爬取子url  ->  external_link将外部链接存进数据库(注：外部链接的判断取决于域名是否相同)  ->  Tb_crawl_data.bulk_insert将爬取的url批量插入数据库  ->  Tb_url_data.bulk_insert将需要审计的url批量存入数据库



### 3、audit审计及指纹识别

1. 注：审计页面会有唯一标识token_id，判断页面是否有被审计，如果已经审计，则不将计入审计；不加载浏览器在/lib/topsec/browser.rb中executable函数中注释掉PhantomJS

1. 流程：/lib/topsec/framework.rb:audit  ->  /lib/topsec/framework/audit.rb  ->  browser_cluster(初始化六个浏览器)  -->  audit_queues(审计队列)  -->  pop_page_for_crawl(从page_queue中获取页面)  -->  audit_page(审计单个页面)  -->  Tb_fingerprint_data.save_extract(存储或更新指纹识别信息)  -->  notify_on_page_audit  -->  http.update_cookies  -->  checks.any?(是否有插件需要检测)  -->  run_checks(执行插件)  -->  check_page(检测页面)  -->  run_one(检测)  -->  /lib/topsec/check/auditor.rb: audit(执行审计，主要是需要注入的插件)  -->  audit_signature(审计特征)                                                            -->   /lib/topsec/element/capabilities/analyzable/signature.rb:  signature_analysis(特征分析)   -->  get_matches(特征值匹配)  -->  find_signatures(特征匹配)  -->  find_signature  -->  control_and_log(日志记录)  -->  @auditor.log(记录日志)  -->  /lib/topsec/check/auditor.rb:log (数据库指纹识别及记录日志)

3. 结论：主要是数据库的指纹识别，会存在误报或者漏报（主要是依靠是否有注入的漏洞）



### 4、指纹识别插件和whatweb

1. 流程：/lib/topsec/page.rb: initialize(Platform::Manager.fingerprint( self ))  -->  /lib/topsec/platform/manager.rb: self.fingerprint  -->  fingerprinters[name].new( page ).run(执行arachni自带的插件和whatweb插件)

1. arachni插件和whatweb插件对比

2. arachni插件识别指纹
	1. 主要是通过headers中的server和x-powered-by、cookies中的键来进行指纹识别的（注：server可能会被隐藏或修改，导致指纹识别漏报或者误报）
	1. 少数通过parserments参数来识别

whatweb插件指纹识别

1. 通过调用web_plugins和language_plugins下的插件，主要是通过regex正则表达式和MD5值匹配（优点，如果正则表达式越多，指纹识别越准确；缺点，需要手动添加）