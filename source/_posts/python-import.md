---
title: __import__详解
date: 2019-04-18 10:53:08
categories:
  - Python
---
1. `__import__`函数的作用

    - 使用import导入python模块时,默认调用的是`__import__`函数.
    - 直接使用该函数,一般用于动态加载模块.

1. `__import__`函数的定义

    在`__builtin__.py`中可以看到`__import__`函数的定义如下:
    `__import__(name, globals={}, locals={}, fromlist=[], level=-1)`

1. `__import__`函数的参数

    由`__import__`函数的定义可以看出, 该函数一共有五个参数, 其中name是必选参数, 其它是可选参数.
    特别要注意:
    - 只传入使用name参数,只能导入最顶层模块(如示例中`In[2]`和`Out[2]`);
    - 同时传入name和fromlist参数(只要list的len不是0即可,根本不care传入的内容),则可以导入子模块(如示例中`In[3]`和`Out[3]`);
    - level参数,指定导入方式（相对导入或者绝对导入,默认两者都支持）
    
    REPL示例:
    ```
    In[1]: __import__('resource_management')
    Out[1]: <module 'resource_management' from '/data/projects/ambari/ambari-lib/2.7.3/resource_management/__init__.py'>
    In[2]: __import__('resource_management.core')
    Out[2]: <module 'resource_management' from '/data/projects/ambari/ambari-lib/2.7.3/resource_management/__init__.py'>
    In[3]: __import__('resource_management.core', fromlist=['notexist'])
    Out[3]: <module 'resource_management.core' from '/data/projects/ambari/ambari-lib/2.7.3/resource_management/core/__init__.py'>
    ```
