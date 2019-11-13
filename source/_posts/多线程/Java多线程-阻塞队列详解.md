---
title: Java多线程 - 阻塞队列详解
date: 2019-06-12 17:19:04
tags:
---

# Java多线程 - 阻塞队列详解

## 定义

- 队列：其中的元素先进先出

- 阻塞：写入队列空间时当队列满会阻塞，获取队列数据时当队列为空时将阻塞。

- 实际情况：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。

### 阻塞队列的继承关系

<center>

<img src="https://cdn.guohezuzi.cn/public/img/blockingqueue.png" width="100%" />

</center>

- ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。

- LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。

- PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。

- DelayQueue：一个使用优先级队列实现的无界阻塞队列。

- SynchronousQueue：一个不存储元素的阻塞队列。

- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

## 常用方法

<center>

<img src="https://cdn.guohezuzi.cn/public/img/blockingqueueMethod.png" width="70%" />

</center>

#### 详解

- add(e) / remove(o) : 阻塞时抛异常，向队列中添加/移除一个指定元素，add操作当队列空间满了，会抛出IllegalStateException异常，当队列空间有限制时，**建议使用offer方法**；remove():向队列移除头部元素，当无元素时，抛出NoSuchElementException 异常(接口Queue的方法)

- offer(e) / poll() :阻塞时返回值， 向队列中添加/取出一个元素，offer操作当队列空间满了，返回false；poll操作，将队列头部元素取出，无元素返回null

- put(e)/take: 阻塞时等待，向队列中添加/删除一个元素，put操作队列满了，进行阻塞等待；take操作队列为空，同样进行阻塞等待。

## 使用Demo

模拟阻塞队列在多线程中使用，主线程中添加元素，另起一个线程消费队列，同时另一个线程打印队列的长度(检测阻塞队列)

```java
public class BlockingQueueDemo {
    // 在主线程向阻塞队列中添加任务，同时开启一个线程消费队列和另一个线程打印队列的长度
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Runnable> blockingQueue = new ArrayBlockingQueue<>(4);
        blockingQueue.put(new BlockingQueueRunnable("run-1"));
        blockingQueue.put(new BlockingQueueRunnable("run-2"));
        blockingQueue.put(new BlockingQueueRunnable("run-3"));
        blockingQueue.put(new BlockingQueueRunnable("run-4"));
        ResumeBlockingQueueThread blockingQueueThread = new ResumeBlockingQueueThread(blockingQueue);
        blockingQueueThread.start();
        GetBlockingQueueSize getBlockingQueueSize = new GetBlockingQueueSize(blockingQueue);
        getBlockingQueueSize.start();

        blockingQueue.put(new BlockingQueueRunnable("run-5"));
        blockingQueue.put(new BlockingQueueRunnable("run-6"));
        blockingQueue.put(new BlockingQueueRunnable("run-7"));
        blockingQueue.put(new BlockingQueueRunnable("run-8"));
        blockingQueue.put(new BlockingQueueRunnable("run-9"));
        blockingQueue.put(new BlockingQueueRunnable("run-10"));
    }

    // 阻塞队列中的线程
    private static class BlockingQueueRunnable implements Runnable {
        public BlockingQueueRunnable(String name) {
            this.name = name;
        }
        private String name;

        @Override
        public void run() {
            System.out.println("thread" + name + ": run");
        }
    }

    // 每隔1s消费阻塞队列的元素个数 10次
    private static class ResumeBlockingQueueThread extends Thread {
        BlockingQueue<Runnable> blockingQueue;

        public ResumeBlockingQueueThread(BlockingQueue<Runnable> blockingQueue) {
            this.blockingQueue = blockingQueue;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    //sleep 1s
                    Thread.sleep(2000);
                    blockingQueue.take().run();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // 每隔2s打印阻塞队列的元素个数 10次
    private static class GetBlockingQueueSize extends Thread {
        BlockingQueue<Runnable> blockingQueue;

        public GetBlockingQueueSize(BlockingQueue<Runnable> blockingQueue) {
            this.blockingQueue = blockingQueue;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("阻塞队列的长度为:"+blockingQueue.size());
            }
        }
    }
}
```

## 使用原理

以ArrayBlockingQueue为例，

需要阻塞时，通过使用Condition类，通过Condition类的await方法进行线程的阻塞。Condition类可参考:[Java多线程 - AQS详解](https://www.guohezuzi.cn/article/java-multithread-aqs#condition)

不需要阻塞时，通过循环数组实现的队列入队/出队

## Ref:

1. JDK8源码

2. [InfoQ 聊聊并发（七）——Java 中的阻塞队列](https://www.infoq.cn/article/java-blocking-queue)

3. [一个自己实现的阻塞队列](https://crossoverjie.top/JCSprout/#/thread/ArrayBlockingQueue)
