---
layout: post
title: 'arachni指纹识别（二）'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

在前一周搭建完环境之后，终于又可以开始学习关于自己需要学习的指纹识别模块



## 指纹识别

- 指纹识别插件如何执行？

  ```ruby
  # 进入指纹识别入口page.rb中
  Platform::Manager.fingerprint( self )
  
  
  # 进入/lib/arachni/platform/manager.rb
  def self.fingerprinters
          @manager ||=
              Component::Manager.new( Options.paths.fingerprinters,
                                      Platform::Fingerprinters )
      end
      fingerprinters.load_all
  
      # @param    [HTTP::Response, Page]  resource
      #
      # @return   [Bool]
      #   `true` if the resource should be fingerprinted, `false` otherwise.
      def self.fingerprint?( resource )
          !(!Options.fingerprint? || resource.code != 200 || !resource.text? ||
              include?( resource.url ) || resource.scope.out?)
      end
  
  # 通过循环依次执行指纹识别插件
  def self.fingerprint( page )
          synchronize do
              # fingerprint?()方法中有一个Options.fingerprint?方法，返回一个bool值
              # Options.fingerprint?返回@no_fingerprinting的值，这个参数是用来决定是否执行指纹识别插件
              # 如果@no_fingerprinting为true时，不执行指纹识别插件，反之
              return page if !fingerprint? page
              
              # fingerprinters是self.fingerprinters函数返回的一个Component::Manager.new对象
              # 通过fingerprinters.load_all导入所有指纹识别插件，并且通过debug调试发现，导入所有的指纹识别插件，是在程序刚开始执行就有了，---------------思考：如何指定指纹识别插件
              fingerprinters.available.each do |name|
                  exception_jail( false ) do
                      # 依次执行指纹识别插件
                      fingerprinters[name].new( page ).run
                  end
              end
  
              # We do this to flag the resource as checked even if no platforms
              # were identified. We don't want to keep checking a resource that
              # yields nothing over and over.
              update( page.url, [] )
          end
  ```

- 问题：如何指定指纹识别插件

  - 思路一

    - 在导入所有插件那里，直接指定指纹识别插件

      ```ruby
      fingerprinters.load_all
      # 更改为
      fingerprints.load(options.platforms)
      ```

    - 需要解决的问题

      如何得到options.platforms，因为导入所有指纹识别插件操作是在程序运行开始的时候执行的

      

  - 思路二

    - 在进行命令行传参的时候导入指纹识别插件，删除程序开始时导入所有指纹识别插件操作

      ```ruby
      # fingerprinters.load_all
      # 在/ui/cli/framework.rb中parse_options函数中增加
      if options.platforms.any?
          begin
              Platform::Manager.new( options.platforms )
              # 增加这行
              Platform::Manager.new.fingerprinters.load(options.platforms)
          rescue Platform::Error::Invalid => e
              options.platforms.clear
              print_error e
              print_info 'Available platforms are:'
              print_info Platform::Manager.new.valid.to_a.join( ', ' )
              print_line
              print_info 'Use the \'--platforms-list\' parameter to see a' <<
                  ' detailed list of all available platforms.'
              exit 1
          end
      end
      ```

    - 遇到的问题

      - 当我导入我们指定的指纹识别插件后，还是执行所有的指纹识别插件

        ```ruby
        def self.fingerprint( page )
                synchronize do
                    return page if !fingerprint? page
                    
                    # fingerprints的值为我们指定的指纹识别插件
                    # fingerprinters.available的值为所有的指纹识别的插件
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
        ```

  

  

## 待解决问题

通过两种思路的比较，可以从第二种思路入手，需要解决相应的问题，指定插件



## 总结

分析问题能力还不够全面，时间的控制也不是很好，一方面对自己的模块还不够熟悉，另一方面学习效率有待提高



欲知后事如何，请看下篇blog分解！！！😏

