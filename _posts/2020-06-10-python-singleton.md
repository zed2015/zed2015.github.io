---
layout:     post
title:      Python thread safe singleton decorator
subtitle:   线程安全的 Python 单例装饰器
date:       2020-06-10
author:     ZC
header-img: 
catalog: 	 true
tags:
    - Python
    - Interview
---

你真的能写好一个单例模式吗？简单的单例模式，大家都会写，装饰器版本的写法、重写__new__版本的写法，但是如何确保线程安全呢？ 下面就简单看一下线程安全的装饰器吧。
talk is cheap, show me the code!
```Python3

def singleton(cls):
    lock = threading.Lock()

    @functools.wraps(cls)
    def inner(*args, **kwargs):
        if not hasattr(cls, "__inst"):
            with lock:
                if not hasattr(cls, "__inst"):
                    inst = cls(*args, **kwargs)
                    setattr(cls, "__inst", inst)

        return getattr(cls, "__inst")

    return inner

```
