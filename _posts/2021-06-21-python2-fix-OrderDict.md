---
layout:     post                    # 使用的布局（不需要改）
title:      python2 OrderDict 循环引用导致频繁gc    # 标题 
subtitle:   ''    #副标题
date:       2021-06-21              # 时间
author:     zc                   # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - python
    - 调优
---

### python2 的OrderDict 导致的内存回收问题
>线上定位到周期性的卡顿，最终定位到 2代GC 导致的卡顿，通过gc 模块排查，顺带发现python2 的 OrderDict 会产生循环引用对象

#### 产生原因
- OrderedDict 有序性采用双向链表存储，正常情况下 需要有__del__ 方法处理双向链表， 早期的python2 版本是这样做的，但是后期发现对于相互引用的OrderedDict 会导致严重的内存泄漏，因此不得不删除__del__方法
- 通过删除__del__ 方法解决了内存泄漏问题，但是会导致产生大量的循环引用对象，触发GC 问题，虽然不影响python2的功能，但是影响了性能；
- 变更历史记录，参考：https://github.com/python/cpython/commit/2039753a9ab9d41375ba17877e231e8d53e17749#diff-52502c75edd9dd62aa7a817dbab542d2

#### 解决办法

```
def clear_2(self):
    root = self._OrderedDict__root
    node = root
    while node[1]:
        node[0] = None
        next_node = node[1]
        node[1] = None
        node = next_node
        
    # root[:] = [root, root, None]
    self._OrderedDict__map.clear()
    dict.clear(self)
OrderedDict.__del__ = clear_2

```

