---
title: python属性管理
date: 2019-04-19 12:41:09
categories:
  - Python
---

在python中,一个类或者类实例中的属性是动态的,也就是说,你可以往一个类或者实例中添加或者删除一个属性.

python管理属性的方法一般有三种:
- 操作符重载(`__getattr__, __setattr__, __delattr__和__getattribute__`)
- property内置函数
- 描述符协议(descriptor)


# 使用操作符重载管理属性
- `__getattr__`方法将拦截所有未定义的属性获取
- `__getattribute__`方法将拦截所有属性的获取
- `__setattr__`方法将拦截所有的属性赋值
- `__delattr__`方法将拦截所有的属性删除

值得注意的是,
由于`__getattribute__, __setattr__, __delattr__`方法均是对所有的属性进行拦截,因而在重载他们的时候,一定要避免递归调用(否则,会引起死循环);
对于`__getattr__`方法,则没有那么多限制.
另外, 操作符重载方法调用不能委托给被包装的对象, 除非包装类自己重新定义这些方法(?).

## 重载`__setattr__`方法
在重载`__setattr__`方法时,不能使用`self.name = value`格式,否则会导致递归调用而陷入死循环,正确的做法应该是:
```
def __setattr__(self, name, value):
    # do something
    object.__setattr__(self, name, value)
    # do something
```
也可以将`object.__setattr__(self, name, value)`换成`self.__dict__[name] = value`,
但是一定要注意如果重载了`__getattribute__`方法,则必须保证`__getattribute__`方法重载正确.
## 重载`__delattr__`方法
在重载`__delattr__`方法时,不能使用`del self.name`格式,否则,它也会导致递归调用而陷入死循环.正确的做法应该是:
```
def __delattr__(self, name):
    # do something
    object.__delattr__(self, name)
    # do something
```
也可以将`object.__delattr__(self, name)`换成`del self.__dict__[name]`,
同样要注意如果重载了`__getattribute__`方法,则必须保证`__getattribute__`方法重载正确.
## 重载`__getattribute__`方法
在重载`__getattribute__`方法时,不能使用`return self.name`格式,否则,也会导致递归调用而陷入死循环.正确的做法是:
```
def __getattribute__(self, name):
    # do something
    value = object.__getattribute__(self, name)
    # do something
    return value
```
特别注意,不能将`object.__getattribute__(self, name)`替换为`self.__dict__[name]`来避免循环,
因为`self.__dict__`也会触发属性获取,所以依然会导致递归调用.
## 重载`__getattr__`方法
没有特殊要求,它的作用一般是用来定义用户获取了类或者类实例中没有定义的属性时,程序应该做出的回应.


# 使用descriptor管理属性
descriptor是一种实现了`__get__`, `__set__`和`__delete__`这三种成员方法中任意一个或多个的特殊类.
下面是示例代码:
```
class RevealAccess(object):
    def __init__(self, val, name):
        self.val = val
        self.name = name

    def __get__(self, obj, cls):
        print('get...', self.name)
        return self.val

    def __set__(self, obj, val):
        print('set...', self.name)
        self.val = val

    def __delete__(self, obj):
        print('delete...')
        del self.val


class MyClass(object):
    x = RevealAccess(0, 'x')


In [1]: m = MyClass()

In [2]: m.x = 1
('set...', 'x')

In [3]: m.x
('get...', 'x')
Out[3]: 1

In [4]: del m.x
delete...

In [5]: m.x
('get...', 'x')
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-7-ed46b668a20d> in <module>()
----> 1 m.x

/data/projects/ambari/ambari-stack/scripts/test.py in __get__(self, obj, cls)
     47     def __get__(self, obj, cls):
     48         print('get...', self.name)
---> 49         return self.val
     50 
     51     def __set__(self, obj, val):

AttributeError: 'RevealAccess' object has no attribute 'val'
```


# 使用property类管理属性(只适用于新式类)
## property类
property类实际上是python内部已经实现好的一种descriptor.在__builtin__.py, 可以看到property类的定义如下(内建类看不到具体实现,可以看到大致模样):
```
class property(object):
    def deleter(self, *args, **kwargs): 
        pass

    def getter(self, *args, **kwargs): 
        pass

    def setter(self, *args, **kwargs): 
        pass
    
    def __delete__(self, obj): # real signature unknown; restored from __doc__
        """ descr.__delete__(obj) """
        pass

    def __get__(self, obj, type=None): # real signature unknown; restored from __doc__
        """ descr.__get__(obj[, type]) -> value """
        pass
        
    def __set__(self, obj, value): # real signature unknown; restored from __doc__
        """ descr.__set__(obj, value) """
        pass
    
    def __getattribute__(self, name): # real signature unknown; restored from __doc__
        """ x.__getattribute__('name') <==> x.name """
        pass

    def __init__(self, fget=None, fset=None, fdel=None, doc=None): # known special case of property.__init__
        """
        property(fget=None, fset=None, fdel=None, doc=None) -> property attribute
        """
        pass

    @staticmethod # known case of __new__
    def __new__(S, *more): # real signature unknown; restored from __doc__
        """ T.__new__(S, ...) -> a new object with type S, a subtype of T """
        pass

    fdel = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default

    fget = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default

    fset = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default
```
## 示例: property实例
说明, property的fget必选,其他两个参数可选
```
class A(object):
    def __init__(self):
        print('init...')
        self._x = 0

    def getx(self):
        print('getx...')
        return self._x

    def setx(self, value):
        print('setx...')
        self._x = value

    def delx(self):
        print('delx...')
        del self._x

    x = property(getx, setx, delx)


In [1]: a = A()
init...

In [2]: a.x = 1
setx...

In [3]: a.x
getx...
Out[3]: 1

In [4]: del a.x
delx...

In [5]: a.x
getx...
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-9-e27186f28a17> in <module>()
----> 1 a.x

/data/projects/ambari/ambari-stack/scripts/test.py in getx(self)
      6     def getx(self):
      7         print('getx...')
----> 8         return self._x
      9 
     10     def setx(self, value):

AttributeError: 'A' object has no attribute '_x'
```
## 示例: property装饰器
说明,可以只有@property
```
class B(object):
    def __init__(self):
        print('init...')
        self._x = 0

    @property
    def x(self):
        print('getx...')
        return self._x

    @x.setter
    def x(self, value):
        print('setx...')
        self._x = value

    @x.deleter
    def x(self):
        print('delx...')
        del self._x


In [1]: b = B()
init...

In [2]: b.x = 1
setx...

In [3]: b.x
getx...
Out[3]: 2

In [4]: del b.x
delx...

In [5]: b.x
getx...
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-10-252ebe2d9b6c> in <module>()
----> 1 b.x

/data/projects/ambari/ambari-stack/scripts/test.py in x(self)
     27     def x(self):
     28         print('getx...')
---> 29         return self._x
     30 
     31     @x.setter

AttributeError: 'B' object has no attribute '_x'
```
