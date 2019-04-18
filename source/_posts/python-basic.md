---
title: python基础知识
date: 2019-04-18 10:53:08
categories:
  - Python
---
# python属性管理
在python中,一个类或者类实例中的属性是动态的,也就是说,你可以往一个类或者实例中添加或者删除一个属性.

python管理属性的方法一般有三种:
- 操作符重载(`__getattr__, __setattr__, __delattr__和__getattribute__`)
- property内置函数
- 描述符协议(descriptor)

## 使用操作符重载管理属性
- `__getattr__`方法将拦截所有未定义的属性获取
- `__getattribute__`方法将拦截所有属性的获取
- `__setattr__`方法将拦截所有的属性赋值
- `__delattr__`方法将拦截所有的属性删除

值得注意的是,
由于`__getattribute__, __setattr__, __delattr__`方法均是对所有的属性进行拦截,因而在重载他们的时候,一定要避免递归调用(否则,会引起死循环);
对于`__getattr__`方法,则没有那么多限制.
另外, 操作符重载方法调用不能委托给被包装的对象, 除非包装类自己重新定义这些方法(?).

### 重载`__setattr__`方法
在重载`__setattr__`方法时,不能使用`self.name = value`格式,否则会导致递归调用而陷入死循环,正确的做法应该是:
```
def __setattr__(self, name, value):
    # do something
    object.__setattr__(self, name, value)
    # do something
```
也可以将`object.__setattr__(self, name, value)`换成`self.__dict__[name] = value`,
但是一定要注意如果重载了`__getattribute__`方法,则必须保证`__getattribute__`方法重载正确.
### 重载`__delattr__`方法
在重载`__delattr__`方法时,不能使用`del self.name`格式,否则,它也会导致递归调用而陷入死循环.正确的做法应该是:
```
def __delattr__(self, name):
    # do something
    object.__delattr__(self, name)
    # do something
```
也可以将`object.__delattr__(self, name)`换成`del self.__dict__[name]`,
同样要注意如果重载了`__getattribute__`方法,则必须保证`__getattribute__`方法重载正确.
### 重载`__getattribute__`方法
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
### 重载`__getattr__`方法
没有特殊要求,它的作用一般是用来定义用户获取了类或者类实例中没有定义的属性时,程序应该做出的回应.

## 使用property内置函数管理属性
在python中,隐式的存在一种类型property类,用来管理类实例的属性访问控制的"特性"正是使用这个property类.
该类可以管理自定义类的实例的属性访问控制.
### property类
property类有三个成员方法: fget, fset, fdel, 它们分别用来管理属性访问;
以及三个装饰器函数: getter, setter, deleter, 它们分别用来把三个同名的类方法装饰成property.
在__builtin__.py, 可以看到property类的定义如下(省略了一部分):
```
class property(object):
    def deleter(self, *args, **kwargs): 
        pass

    def getter(self, *args, **kwargs): 
        pass

    def setter(self, *args, **kwargs): 
        pass
    
    # ...

    fdel = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default

    fget = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default

    fset = property(lambda self: object(), lambda self, v: None, lambda self: None)  # default
```

## python中的descriptor
1. descriptor是对象的一个属性;
1. descriptor必须存在于类的__dict__中;
1. descriptor一定有__get__特殊方法,也可以有__set__,__delete__特殊方法.
实现了某些特定方法的对象,其特定方法是:__get__,__set__,__delete__,其中__set__和__delete__方法是可选的.
1. descriptor必须依附于对象,作为对象的一个属性,不能单独存在.
1. descriptor必须存在于类的__dict__中,也就是说只有在类的__dict__中找到属性,python才会去看它有没有__get__等方法,对一个在实例的__dict__中找到的属性,python根本不理会其是否有__get__方法,直接返回属性本身.

