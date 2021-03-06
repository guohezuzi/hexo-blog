---
title: Java static加载机制
date: 1999-02-24
tags:
---

# Java static加载机制

## 静态变量

- 在类加载时，将静态存储结构转换为方法区的运行时数据结构

- 在类初始化时，进行初始化

## 静态代码块

- 类初始化时加载

## 静态内部类

- 外部类被加载时内部类并不会立即加载内部类，内部类类加载时进行加载

## 类加载过程

![](https://images2015.cnblogs.com/blog/879896/201604/879896-20160414224549770-60006655.png)

### 类加载时机

类的加载是通过类加载器（Classloader）完成的，它既可以是饿汉式[eagerly load]（只要有其它类引用了它就加载）加载类，也可以是懒加载[lazy load]（等到类初始化发生的时候才加载）。不过我相信这跟不同的JVM实现有关，然而他又是受JLS保证的（当有静态初始化需求的时候才被加载）。

### 初始化时机

- 使用new关键字实例化对象的时候、读取或设置一个类的静态字段的时候，已经调用一个类的静态方法的时候。

- 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有初始化，则需要先触发其初始化。

- 当初始化一个类的时候，如果发现其父类没有被初始化就会先初始化它的父类。

- 当虚拟机启动的时候，用户需要指定一个要执行的主类（就是包含main()方法的那个类），虚拟机会先初始化这个类；

- 使用Jdk1.7动态语言支持的时候的一些情况。
