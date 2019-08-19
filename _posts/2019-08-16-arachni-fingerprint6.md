---
layout: post
title: 'arachni指纹识别（六）'
categories: arachni
tags: arachni ruby linux work
author: thorleishen
---

* content
{:toc}
## 前言

研究check.supports_platforms?( ps )）如何影响插件的执行



## 插件中指纹的判断

通过对比插件中是否含有我们指定的指纹或者指纹插件识别出来的指纹来决定该插件是否支持该指纹的识别，如果支持则执行该插件，反之，直接return跳过，代码如下：

```ruby
def supports_platforms?( resource_platforms )
    if resource_platforms.any? && has_exempt_platforms?
        manager = Platform::Manager.new( exempt_platforms )

        resource_platforms.each do |p|

            # When we check for exempt platforms we're looking for info
            # from the same type.
            ptype = Platform::Manager.find_type( p )
            type_manager = manager.send( ptype )

            return false if type_manager.pick( p => true ).any?
        end
    end

    return true if resource_platforms.empty? || !has_platforms?

    # Determine if we've got anything for the given platforms, the same
    # way payloads are picked.
    foo_data = self.platforms.
        inject({}) { |h, platform| h.merge!( platform => true ) }

    Platform::Manager.new( resource_platforms ).pick( foo_data ).any?
end

# if resource_platforms.any? && has_exempt_platforms?，大多数都为false，这里并不做强调，意义不大
# resource_platforms.any?判断我们是否指定指纹或指纹插件是否识别指纹
# has_exempt_platforms判断漏扫插件中self.info中是否含有exempt_platforms参数

# return true if resource_platforms.empty? || !has_platforms?
# has_platforms?判断漏扫插件中self.info中是否含有platforms插件
# 这里也是我们区分带有指纹插件和不带有指纹插件，不带有指纹插件，则必返回true，那么该插件一定会执行；反之，带有指纹的插件会经过Platform::Manager.new( resource_platforms ).pick( foo_data ).any?的判断
if !check.supports_platforms?( ps )
    print_info "Check #{check.shortname} does not support: #{ps.join( ' + ' )}"
    return false
end

# foo_data = self.platforms.inject({}) { |h, platform| h.merge!( platform => true ) }
# 将漏扫插件中self.info中的platforms转成hash值，例：{:php => true}

# Platform::Manager.new( resource_platforms ).pick( foo_data ).any?
# Platform::Manager.new( resource_platforms )返回一个Platform::Manager的实力对象，可通过to_a方法返回指定的指纹
# pick( foo_data )
def pick( data_per_platform )
    data_per_list = {}
    data_per_platform.each do |platform, value|
        list = find_list( platform )
        data_per_list[list]           ||= {}
        data_per_list[list][platform]   = value
    end

    picked = {}
    data_per_list.each do |list, data|
        # If a platform list is empty pass the given data without picking...
        if list.empty?
            picked.merge! data
            next
        end

        # ...otherwise enforce its platform restrictions.
        picked.merge! list.pick( data )
    end

    picked
end

# find_list( platform )，获取指纹对应的类别List对象，主要包含platforms指定的指纹，和vail_platforms支持的指纹，将漏扫插件中的指纹放入data_per_list的platform中
# if list.empty?判断该类型的指纹中是否有我们指定的指纹之一，如果有则不为空返回false，执行picked.merge! list.pick( data )
def pick( data_per_platform )
    orig_data_per_platform = data_per_platform.dup
    data_per_platform      = data_per_platform.dup

    data_per_platform.select! { |k, v| include? k }

    # Bail out if the valid platforms are just a flat array, without hierarchy.
    return data_per_platform if !hierarchical?

    # Keep track of parents which will be removed due to the existence of
    # their children.
    specified_parents = []

    # Remove parents if we have children.
    data_per_platform.keys.each do |platform|
        specified_parents |= parents = find_parents( platform )
        data_per_platform.reject! { |k, _| parents.include? k }
    end

    # Include all of the parents' children if parents are specified but no
    # children for them.

    children = {}
    children_for = valid & @platforms.to_a
    children_for.each do |platform|
        next if specified_parents.include? platform
        c = find_children( platform )
        children.merge! orig_data_per_platform.select { |k, _| c.include? k }
    end

    data_per_platform.merge! children

    # Include the nearest parent data there is a child platform but there
    # are no data for it.

    ignore = data_per_platform.keys | specified_parents
    orig_data_per_platform.each do |platform, data|
        next if ignore.include?( platform ) ||
            !include_any?( find_children( platform ) )
        data_per_platform[platform] = data
    end

    data_per_platform
end
# 该函数主要是判断data中的指纹之一是否在对应的指纹类别中platforms中存在，存在则返回相应的指纹；反之，返回空数组EmptyArray
# Platform::Manager.new( resource_platforms ).pick( foo_data ).any?中Platform::Manager.new( resource_platforms ).pick( foo_data )有值返回true，反之
# 则check.supports_platforms?( ps )，有值返回true，即执行插件；反之，不支持该指纹识别，会return false，不会执行插件
```

![皮皮怪](https://raw.githubusercontent.com/Thorleishen/thorleishen.github.io/master/images/2019-08-16-img1.png)

## 总结

通过这几天的指纹的学习，已经深入了解指纹对漏扫插件的影响，即指纹会影响一些插件的执行，这些插件就是self.info中带有platforms的插件，故之后会将移植的插件进行整理，完善漏扫插件中self.info中platforms的信息，希望自己这几天的学习对今后的学习有所帮助