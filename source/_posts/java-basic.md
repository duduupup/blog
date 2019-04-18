---
title: java基础知识
date: 2019-04-09 09:09:14
categories:
  - Java
---
# 静态类
1. java中普通的顶级类不能使用static关键字修饰,只有内部类可以使用static修饰,但是内部类并不一定要用static修饰
1. 普通内部类与静态内部类的区别

    1. 普通内部类保有对外部类的引用,所以普通内部类的实例不能够单独存在,必须通过外部类的实例来实例化.
    同时,普通内部类可以直接使用外部类的所有的属性和方法(包括私有的以及静态的属性和方法).
    普通内部类的实例化如下:
    Users users = new Users();
    Users.CommonClass commonClass = users.new CommonClass();
    
    1. 静态内部类不保有对外部类的引用,因而静态内部类的实例可以单独存在.
    同时,静态内部类也就只能访问外部类的静态属性和方法.
    内部静态类的实例化如下:
    StaticClass staticClass = new StaticClass();
    Users.StaticClass staticClass2 = new Users.StaticClass(); // 推荐使用此方法
    
# 