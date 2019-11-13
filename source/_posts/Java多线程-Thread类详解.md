---
title: Java多线程-Thread类详解
date: 2019-08-30
tags:
---

# Java多线程-Thread类详解

## 实例方法

2. 对线程对象的一些局部变量的get/set方法，id、name、priority...

3. 线程对象的一些状态信息，getState()、isAlive()、isDaemon()...

## 静态方法

### yield()方法

A hint to the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this hint.
Yield is a heuristic attempt to improve relative progression between threads that would otherwise over-utilise a CPU. Its use should be combined with detailed profiling and benchmarking to ensure that it actually has the desired effect.
It is rarely appropriate to use this method. It may be useful for debugging or testing purposes, where it may help to reproduce bugs due to race conditions. It may also be useful when designing concurrency control constructs such as the ones in the java.util.concurrent.locks package.

向调度程序提示当前线程是否愿意让度其当前使用的处理器。 调度程序可以忽略此提示。
Yield是一种启发式尝试，用于改善线程之间的相对进展，否则会过度利用CPU。 它的使用应与详细的分析和基准测试相结合，以确保它实际上具有所需的效果。
使用这种方法很少是合适的。 它可能对调试或测试目的很有用，它可能有助于重现因竞争条件而产生的错误。 在设计并发控制结构（例如java.util.concurrent.locks包中的结构）时，它也可能很有用。

### sleep()方法

Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds, subject to the precision and accuracy of system timers and schedulers. The thread does not lose ownership of any monitors.

导致当前正在执行的线程休眠（暂时停止执行）指定的毫秒数，具体取决于系统计时器和调度程序的精度和准确性。 该线程不会失去任何监视器的所有权(比如对一个资源的锁)

### interrupt()方法

Interrupts this thread.
Unless the current thread is interrupting itself, which is always permitted, the checkAccess method of this thread is invoked, which may cause a SecurityException to be thrown.
If this thread is blocked in an invocation of the wait(), wait(long), or wait(long, int) methods of the Object class, or of the join(), join(long), join(long, int), sleep(long), or sleep(long, int), methods of this class, then its interrupt status will be cleared and it will receive an InterruptedException.
If this thread is blocked in an I/O operation upon an InterruptibleChannel then the channel will be closed, the thread's interrupt status will be set, and the thread will receive a java.nio.channels.ClosedByInterruptException.
If this thread is blocked in a java.nio.channels.Selector then the thread's interrupt status will be set and it will return immediately from the selection operation, possibly with a non-zero value, just as if the selector's wakeup method were invoked.
If none of the previous conditions hold then this thread's interrupt status will be set.
Interrupting a thread that is not alive need not have any effect.

中断当前线程，如果当前线程遇到当前线程正在中断，或者遇到wait()、join()、sleep()阻塞或IO阻塞时会报不同异常。

## 其他

### wait()方法

它是object类的方法，wait()方法需要和notify()及notifyAll()两个方法一起使用，这三个方法用于协调多个线程对共享数据的存取，所以必须在synchronized语句块内使用，也就是说，调用wait()，notify()和notifyAll()的任务在调用这些方法前必须拥有对象的锁。注意，它们都是Object类的方法，而不是Thread类的方法。
　　wait()方法与sleep()方法的不同之处在于，wait()方法会释放对象的“锁标志”。当调用某一对象的wait()方法后，会使当前线程暂停执行，并将当前线程放入对象等待池中，直到调用了notify()方法后，将从对象等待池中移出任意一个线程并放入锁标志等待池中，只有锁标志等待池中的线程可以获取锁标志，它们随时准备争夺锁的拥有权。当调用了某个对象的notifyAll()方法，会将对象等待池中的所有线程都移动到该对象的锁标志等待池。
　　除了使用notify()和notifyAll()方法，还可以使用带毫秒参数的wait(long timeout)方法，效果是在延迟timeout毫秒后，被暂停的线程将被恢复到锁标志等待池。
　　此外，wait()，notify()及notifyAll()只能在synchronized语句中使用，但是如果使用的是ReenTrantLock实现同步，该如何达到这三个方法的效果呢？解决方法是使用ReenTrantLock.newCondition()获取一个Condition类对象，然后Condition的await()，signal()以及signalAll()分别对应上面的三个方法。

## 参考

[sleep()，wait()，yield()和join()方法的区别](https://blog.csdn.net/xiangwanpeng/article/details/54972952)


