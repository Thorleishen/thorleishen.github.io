---
layout: post
title: 'arachni指纹识别（四）'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

需求分析：arachni原生web UI界面中fingerprint配置项和漏洞插件的关联关系

注：文章出现漏洞插件为arachni中components下checks目录下的插件，指纹识别插件为components下fingerprinters目录下的插件



## 研究结果分析

1. arachni在审计过程中，有两种检测方式，第一种是带有fingerprint，另一种则没有带有fingerprint，代码如下：

```ruby
# /lib/arachni/framework/parts: audit_page函数中
run_http = run_checks( @checks.without_platforms, page )
run_http = true if run_checks( @checks.with_platforms, page )

# without_platforms和with_platforms分别过滤带有指纹的和不带指纹的漏洞插件
# @return   [Hash]
#   Checks targeting specific platforms.
def with_platforms
    select { |k, v| v.has_platforms? }
end

# @return   [Hash]
#   Platform-agnostic checks.
def without_platforms
    select { |k, v| !v.has_platforms? }
end

def has_platforms?
    platforms.any?
end
def platforms
    @platforms ||= [info[:platforms]].flatten.compact
end

# 通过上述代码可见，检测一个漏洞插件中是否含有指纹，通过判断漏洞插件中info信息函数中含有platforms参数
```

![皮皮虾](https://github.com/Thorleishen/thorleishen.github.io/raw/master/images/2019-08-08-img1.jpg)

2. 进入run_checks中check_page审计页面时

```ruby
def check_page( check, page )
    ps = page.platforms.to_a

    # If we've been given platforms which the check doesn't support don't
    # even bother running it.
    if !check.supports_platforms?( ps )
        print_info "Check #{check.shortname} does not support: #{ps.join( ' + ' )}"
        return false
    end

    begin
        @checks.run_one( check, page )
    rescue => e
        print_error "Error in #{check.to_s}: #{e.to_s}"
        print_error "Page: #{page.dom.url}"
        print_error_backtrace e
        false
    end
end

# ps = page.platforms.to_a，这里通过Platform::Manager[url]
# 获取指纹识别插件已经在该页面识别出来的指纹

# check.supports_platforms?( ps )判断该漏洞插件是否支持指纹识别插件识别出来的指纹，具体实现可以通过debug边调试边看结果
```



## 总结

希望在之后的时间里，尽早掌握指纹识别与漏洞插件之间的关联并完成需求！！！😂