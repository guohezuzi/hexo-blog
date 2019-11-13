---
title: Java多线程 - AQS详解
date: 2019-06-10 17:19:04
tags:
---

# Java多线程 - AQS详解

## 介绍

AQS是java.util.concurrent.locks下类AbstractQueuedSynchronizer的简称，是用于    通过Java源码来构建多线程的锁和同步器的一系列框架，用于Java多线程之间的同步，它的类及类结构图如下:

<center>    
<img src="https://cdn.guohezuzi.cn/public/img/aqs-class.png" width="60%" /> 
</center>

<center>
 <img src="https://cdn.guohezuzi.cn/public/img/aqs-diagram.png" width="60%" />
 </center>

## 原理

在AQS类中维护了一个使用双向链表Node实现的FIFO队列，用于保存等待的线程，同时利用一个int类型的state来表示状态，使用时通过继承AQS类并实现它的acquire和release方法来操作状态，来实现线程的同步。

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

再以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

## 不同组件的使用

### CountDownLatch

主要用于等待线程等待其他线程执行后再执行，其实现是通过控制计数器是否递减到0来判别，其他的每一个线程执行完毕后，调用countDown()方法让计数器减一，等待线程调用await()方法，直到计数器为1在执行。

demo 主线程等待200个线程执行完毕后再执行：

```java
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: guohezuzi
 * \* Date: 2019-06-08
 * \* Time: 下午4:14
 * \* Description: ContDownLatch用法：通过引入CountDownLatch计数器,来等待其他线程执行完毕
 * \
 */
@Slf4j
public class CountDownLatchExample {
    private static int threadCount = 200;

    public static void test(int threadNum) throws InterruptedException {
        Thread.sleep(100);
        log.info("{}",threadNum);
        Thread.sleep(100);
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool= Executors.newCachedThreadPool();

        final CountDownLatch countDownLatch=new CountDownLatch(200);
        for (int i = 0; i < threadCount; i++) {
            final int threadNum=i;
            pool.execute(()->{
                try {
                    Thread.sleep(1);
                    test(threadNum);
                }catch (Exception e){
                    log.error("exception",e);
                }finally {
                    countDownLatch.countDown();
                }
            });
        }
        countDownLatch.await();
        log.info("finish");
        pool.shutdown();
    }
}
```

### CyclicBarrier

用于等待多个线程都准备好再进行，每一个线程准备好后，计数器加1，加到指定值后全部开始

demo 一个20个线程每等待5个线程进行一次

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.*;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: guohezuzi
 * \* Date: 2019-06-08
 * \* Time: 下午5:20
 * \* Description:
 * 用于等待多个线程都准备好
 * 每一个线程准备好后 计数器加1 加到指定值后全部开始
 * \
 */
public class CyclicBarrierExample {
    private static final Logger logger = LoggerFactory.getLogger(CountDownLatchExample.class);
    private static CyclicBarrier cyclicBarrier=new CyclicBarrier(5);

    public static void race(int threadNum) throws InterruptedException{
        Thread.sleep(1000);
        logger.info("{} is ready",threadNum);
        try {
            //等待指定数量的其他线程执行 无参一直等待不抛异常 有参数表示等待指定时间若数量还未等到抛出异常
            cyclicBarrier.await(2000, TimeUnit.MILLISECONDS);
        } catch (BrokenBarrierException | TimeoutException e) {
            logger.error("exception",e);
        }
        logger.info("{} is continue");
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService= Executors.newCachedThreadPool();
        for (int i = 0; i < 20; i++) {
            Thread.sleep(1000);
            final int threadNum=i;
            executorService.execute(() -> {
                try {
                    race(threadNum);
                } catch (InterruptedException e) {
                    logger.error("exception",e);
                }
            });
        }
        executorService.shutdown();
    }

}
```

### Semaphore

英译信号量，用于控制某个资源同时可被访问的个数，如控制数据库资源可以同时并发数量为20

demo:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.*;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: guohezuzi
 * \* Date: 2019-06-08
 * \* Time: 下午3:39
 * \* Description: 信号量学习例子 控制某个资源同时可被访问的个数 如控制数据库资源可以同时并发数量为20
 * \
 */
public class SemaphoreExample {
    private static final Logger logger = LoggerFactory.getLogger(CountDownLatchExample.class);
    private static int threadCount = 200;

    public static void test(int threadNum) throws InterruptedException {
        Thread.sleep(100);
        logger.info("{}",threadNum);
        Thread.sleep(1000);
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool= Executors.newCachedThreadPool();
        //定义允许并发的信号量m
        final Semaphore semaphore=new Semaphore(20);
        for (int i = 0; i < threadCount; i++) {
            final int threadNum=i;
            //该线程的最大并发数为m/n
            pool.execute(()->{
                try {
                    //获取n个信号量 无参为一个
                    semaphore.acquire(4);
                    test(threadNum);
                    //释放n个信号量 无参为一个
                    semaphore.release(4);
                }catch (Exception e){
                    logger.error("exception",e);
                }
            });
        }
        pool.shutdown();
    }
}
```

### ReentrantReadWriteLock

读写锁，用于需要同步资源时在前后加锁/解锁，当一个线程获取读锁后其他线程可以继续获取读锁，当一个线程获取写锁后其他线程都需等待，因此，可能造成写锁饥饿，就是写锁一直无法获取。

demo: 一个基于aqs锁实现的部分线程安全的map

```java
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: guohezuzi
 * \* Date: 2019-06-08
 * \* Time: 下午11:58
 * \* Description: 读写锁 当一个线程获取读锁后其他线程可以继续获取读锁 当一个线程获取写锁后其他线程都需等待
 * \
 */
public class ReentrantReadWriteLockExample {
    final Map<String, Data> map = new TreeMap<>();

    private final static ReentrantLock lock = new ReentrantLock();

    private final ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private final Lock readLock = readWriteLock.readLock();

    private final Lock writeLock = readWriteLock.writeLock();

    public Data get(String key) {
        readLock.lock();
        try {
            return map.get(key);
        } finally {
            readLock.unlock();
        }
    }

    public Set<String> getAllkeys() {
        readLock.lock();
        try {
            return map.keySet();
        } finally {
            readLock.unlock();
        }
    }

    public Data put(String key, Data vlaue) {
        writeLock.lock();
        try {
            return map.put(key, vlaue);
        } finally {
            writeLock.unlock();
        }
    }

    class Data {

    }
}
```

### StampLock

类似读写锁的功能和使用方法，不过有以下两点不同

1. 每次获取锁会得到一个long类型的stamp所为返回值，解锁是需要将其回传。

2. 有乐观读操作，适合于读多写少情况，指当资源被读锁锁定时，会根据资源是否被变更，进行读取操作，而不是不允许读操作。

demo:

```java
import java.util.concurrent.locks.StampedLock;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: guohezuzi
 * \* Date: 2019-06-09
 * \* Time: 下午1:08
 * \* Description:
 * 使用是每次获取锁会得到一个long类型的stamp所为返回值，解锁是需要将其回传
 * 该类有 写 读 乐观读：指当资源被读锁锁定时，会根据资源是否被变更，进行读取操作
 */
public class StampLockExample {
    private int count = 0;
    private final StampedLock lock = new StampedLock();

    class AddHundredNum extends Thread {
        @Override
        public void run() {
//            synchronized (addHundredNum.class) {
            long stamp = lock.writeLock();
            try {
                for (int i = 0; i < 1000; i++) {
                    count++;
                }
            } finally {
                lock.unlock(stamp);
            }
//            }
        }
    }

    public void test() throws InterruptedException {
        StampLockExample.AddHundredNum[] addHundredNums = new StampLockExample.AddHundredNum[100];
        for (int i = 0; i < addHundredNums.length; i++) {
            addHundredNums[i] = new StampLockExample.AddHundredNum();
        }

        for (StampLockExample.AddHundredNum addHundredNum : addHundredNums) {
            addHundredNum.start();
        }

        for (StampLockExample.AddHundredNum addHundredNum : addHundredNums) {
            addHundredNum.join();
        }
    }

    public static void main(String[] args) throws Exception {
        StampLockExample example = new StampLockExample();
        example.test();
        System.out.println(example.count);
    }
}
```

### Condition

配合AQS锁实现的线程中断/等待机制，将等待的线程移入condition维护的队列，并通过condition控制中断/等待。

```java
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: guohezuzi
 * \* Date: 2019-06-09
 * \* Time: 下午1:26
 * \* Description:
 * \
 */
@Slf4j
public class ConditionExample {
    public static void main(String[] args){
        final ReentrantLock lock=new ReentrantLock();
        Condition condition=lock.newCondition();
        new Thread(()->{
            lock.lock();
            log.info("wait signal");
            try {
                condition.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info("get signal");
            lock.unlock();
        }).start();

        new Thread(() -> {
            lock.lock();
            log.info("get lock");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            condition.signalAll();
            log.info("send signal ~");
            lock.unlock();
        }).start();
    }
}
```

### 
