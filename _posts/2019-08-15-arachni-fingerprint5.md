---
layout: post
title: 'arachni指纹识别（五）'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

通过今天的学习，对arachni中指定插件对漏洞插件的影响有了更深的理解，如下分享今天对其认识



## --platforms-no-fingerprinting和--platforms命令行参数的选择

- --platforms-no-fingerprinting命令行

前面已经介绍，该命令控制是否执行指纹识别插件，具体请参考：

[arachni指纹识别（三）]: https://thorleishen.github.io/2019/08/13/arachni-fingerprint3/



- --platforms命令行参数

该命令是配合--platforms-no-fingerprinting命令行参数的使用，作用是指定指纹识别插件，并且会影响插件的运行，当你看到这篇文章时会有更深的认识，前几篇文章可能所讲内容有些疑问，该文章可得到解释；还是从执行插件入口开始/lib/arachni/framework/check.rb

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
# 由page.platforms.to_a进入
def platforms
    Platform::Manager[url]
end
# Platform::Manager[url]中的重写了[]方法
def self.[]( uri )
    # If fingerprinting is disabled there's no point in filling the cache
    # with the same object over and over, create an identical one for all
    # URLs and return that always.
    if !Options.fingerprint?
        return @default ||= new_from_options
    end

    return new_from_options if !(key = make_key( uri ))
    synchronize { @platforms.fetch(key) { new_from_options } }
end
# 由上述代码可知，Options.fingerprint?判断是否执行指纹识别插件，因此返回@default ||= new_from_options
# Options.platforms为--platforms指定的指纹
def self.new_from_options( platforms = [] )
    new( platforms | Options.platforms )
end

# check.supports_platforms?( ps )判断该漏洞插件是否支持指纹识别插件识别出来的指纹，具体实现可以通过debug边调边查看
```

![皮皮狗](https://raw.githubusercontent.com/Thorleishen/thorleishen.github.io/master/images/2019-08-15-img1.png)

并且指定指纹会影响识别出的指纹，指纹识别打印消息在/lib/arachni/framework/audit.rb：audit_page，审计页面的时候打印

```ruby
if page.platforms.any?
print_info "Identified as: #{page.platforms.to_a.join( ', ' )}"
end
# 由之前ps = page.platforms.to_a，这里可以知道，打印的消息永远时我们指定的指纹
```



## 总结

今天所了解并不代表是我就真正的了解，可能明天再来看的时候会有更加深刻的认识，今天主要是将两个命令行参数的是如何使用的。



欲知指定指纹如何影响插件的执行（即check.supports_platforms?( ps )），请听下回分解！！！😇