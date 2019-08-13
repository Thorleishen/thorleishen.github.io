---
layout: post
title: 'arachni指纹识别（三）'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

继续昨天指定指纹识别插件问题讨论，主要是解决指定指纹识别插件后，还是执行所有的指纹识别插件



## 问题：不管我是否指定指纹识别插件，一直执行所有指纹识别插件

```ruby
def self.fingerprint( page )
    synchronize do
        return page if !fingerprint? page

        fingerprinters.available.each do |name|
            exception_jail( false ) do
                fingerprinters[name].new( page ).run
            end
        end

        # We do this to flag the resource as checked even if no platforms
        # were identified. We don't want to keep checking a resource that
        # yields nothing over and over.
        update( page.url, [] )
    end
    
    # Fingerprinting will have resulted in element parsing, clear the element
    # caches to keep RAM consumption down.
    page.clear_cache
end

# fingerprinters为Component::Manager的实例变量
def self.fingerprinters
    @manager ||=
        Component::Manager.new( Options.paths.fingerprinters,
            Platform::Fingerprinters )
end

# fingerprinters.available获取所有的指纹识别插件
def available
    paths.map { |path| path_to_name( path ) }
end

def paths
    @paths_cache ||=
        Dir.glob( File.join( "#{@lib}**", "*.rb" ) ).
        reject{ |path| helper?( path ) }
end
```

从代码中可以看出，指纹识别的插件执行和fingerprints.available有关，因此我们应该从这里入手；

修改策略：

```ruby
fingerprinters.each do |name|
    exception_jail( false ) do
        fingerprinters[name].new( page ).run
    end
end
# fingerprinters在我们命令行指定指纹识别插件时，导入的指定的指纹识别插件
# 通过我们debug看出来，fingerprinters可以通过迭代函数将我们指定的插件迭代出来，并且是以数组的形式，第个元素为插件名，后一个为插件类名，因此我们可以通过此办法解决指定指纹识别插件的问题
```



## 问题二：对于我们移植的指纹识别插件

比如，现有的whatweb指纹识别插件，我想的办法是使用命令行传参，将现有的指纹识别插件和移植的指纹识别插件区别开来，具体代码如下：

```ruby
# 在/ui/cli/framework.rb中的platform函数添加如下代码
on( '--platforms-whatweb PLATFORM,PLATFORM2,...',
    'Comma separated list of platforms (by shortname) to audit.',
    '(The given platforms will be used *in addition* to fingerprinting. In order to restrict the audit to',
    "these platforms enable the '--platforms-no-fingerprinting' option.)"
    ) do |platforms|
        options.platforms_whatweb |= platforms.split( ',' )
    end
```



## 总结

通过之前和这两天对arachni指纹识别的学习，进一步熟悉arachni指纹识别流程，并且对之后指纹识别模块的维护有很大的帮助；指纹识别略我千百遍，我对指纹识别如初恋！！！👽