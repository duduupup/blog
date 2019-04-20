---
title: python元类
date: 2019-04-19 15:50:30
categories:
  - Python
---
元类,顾名思义就是生成类的类.type是比较特殊的元类,一般情况下,没有指定类的元类,就是用type创建的.
# type
在`__builtin__.py`中type的定义如下:
```
class type(object):
    """
    type(object) -> the object's type
    type(name, bases, dict) -> a new type
    """
    def __init__(cls, what, bases=None, dict=None): # known special case of type.__init__
        """
        type(object) -> the object's type
        type(name, bases, dict) -> a new type
        # (copied from class doc)
        """
        pass
     
    # ...
```
使用type创建一个类,具体如下:
```
In [1]: def fn(self, name='world'):
   ...:     print('Hello, %s!' % name)
   ...:     

In [2]: Hello = type('Hello', (object,), dict(say_hello=fn))

In [3]: h = Hello()

In [4]: type(h)
Out[4]: __main__.Hello

In [5]: h.say_hello()
Hello, world!
```
# `__new__`和`__init__`
可以理解为: 
- `__init__`函数类似与c++或java的构造函数,用来负责类的实例化;
- `__new__`是在构造函数
# 自定义元类
自定义元类必须继承于type