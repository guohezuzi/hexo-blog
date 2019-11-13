---
title: Java多线程-线程安全的单例模式的实现
date: 2019-06-08 17:19:04
tags:
---

# Java多线程-线程安全的单例模式的实现

## 加锁的实现

### 简易版

直接加锁无判断来同步类的创建

```java
public class Singleton0 {
    private Singleton0(){}
    private static Singleton0 instance;
    public static Singleton0 getInstance(){
        synchronized (Singleton0.class){
            if (instance!=null){
                instance=new Singleton0();
            }
        }
        return instance;
    }
}
```

缺点：在多个线程调用`getInstance`方法时，都会加锁后判断，效率低，时间较长。

### 进阶版

在进行同步之前，会对是否对象已经创建进行判断，减少`syschronized`过程

```java
public class Singleton1 {
    private volatile static Singleton1 uniqueInstance;
    private Singleton1() { }
    public static Singleton1 getInstance() {
        if (uniqueInstance == null) {
            synchronized(Singleton0.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton1();
                }
            }
        }
        return uniqueInstance;
    }
}
```

##### 为什么uniqueInstance变量要用volatele关键字？

防止new对象时，发生指令重排

uniqueInstance = new Singleton(); 这段代码其实是分为三步执行：

1. 为 uniqueInstance 分配内存空间
2. 初始化 uniqueInstance
3. 将 uniqueInstance 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

使用 volatile 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

## 利用final关键字的实现

final关键字限制了实例对象的唯一

```java
public class Singleton {
    private Singleton(){}
    private static final Singleton INSTANSE =new Singleton();
    public static Singleton getInstance() {
        return INSTANSE;
    }
}
```

## 利用final+静态内部类实现

优点：

1. 通过内部类，实现更好的封装
2. 在一定程度上实现了懒加载，当外部类加载时，内部类不会加载，不过一般我们在使用，不会加载外部类，而是直接调用`getInstance`方法

```java
public class Singleton {
    private Singleton (){}
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

}
```
