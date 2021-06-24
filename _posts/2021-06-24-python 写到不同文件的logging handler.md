---
layout:     post                    # 使用的布局（不需要改）
title:      python logging               # 标题 
subtitle:   python 多后缀文件的logging handler #副标题
date:       2021-06-24             # 时间
author:     zc                   # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - python
    - multiprocessing
    
---

#### 直接看代码
``` python
class MulitSuffixFileDistributeHandler(Handler):
    """
    support file_suffix in extra param passed into logger._log func
    distribute logs to diffrence file by file_suffix in extra param
    """
    def __init__(self, filename, *args, **kwargs):
        """init"""
        # must init 
        Handler.__init__(self)
        self.args = args
        self.kwargs = kwargs
        self.baseFilename = filename
        self.formatter = None
        self.suffix_handlers = {}

    def createLock(self):
        """
        do not create lock
        """
        self.lock = None
    
    def get_suffix_handler(self, suffix=''):
        """
        get handler by suffix
        """
        if suffix:
            suffix_filename = self.baseFilename + '-' + suffix
        else:
            suffix_filename = self.baseFilename
        handler = MultiprocessSafeTimedRotatingFileHandler(filename=suffix_filename, *self.args, **self.kwargs)
        handler.setFormatter(self.formatter)
        handler.filters = self.filters
        handler.setLevel(self.level)
        return handler
    
    def handle(self, record):
        """
        handle log record
        """
        file_suffix = getattr(record, 'file_suffix', '')
        if file_suffix in self.suffix_handlers:
            handler = self.suffix_handlers[file_suffix]
        else:
            handler = self.get_suffix_handler(suffix=file_suffix)
            self.suffix_handlers[file_suffix] = handler
        
        handler.handle(record)
    
    def close(self):
        """close"""
        Handler.close(self)
        for k, handler in self.suffix_handlers.items():
            handler.close()
```
