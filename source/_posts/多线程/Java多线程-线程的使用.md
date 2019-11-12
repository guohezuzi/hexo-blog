# Java多线程 - 线程的使用

## 线程的创建

#### 通过实现Runnable接口

```java
class RunnableDemo implements Runnable {
    private String threadName;

    private RunnableDemo(String name) {
        this.threadName = name;
        System.out.println("creating thread:" + threadName);
    }

    @Override
    public void run() {
        System.out.println("Running " + threadName);

        try {
            for (int i = 0; i < 10; i++) {
                System.out.println("Thread:" + threadName + "," + i);
                Thread.sleep(50);
            }
        } catch (InterruptedException e) {
            System.out.println("Thread " + threadName + "interrupter");
        }
        System.out.println("Thread " + threadName + " exiting");
    }
    // run
    public static void main(String[] args) {
        RunnableDemo r = new RunnableDemo("MyThread");
        r.run();
    }
}
```

#### 通过继承Thread类本身

```java
public class ThreadDemo extends Thread {
    @Override
    public void run() {
        System.out.println("thread" + Thread.currentThread().getId() + " running...");
    }
    // run 10 thread
    public static void main(String[] args) throws InterruptedException {
        ThreadDemo[] threadDemos = new ThreadDemo[10];
        for (int i = 0; i < threadDemos.length; i++) {
            threadDemos[i] = new ThreadDemo();
        }
        for (ThreadDemo threadDemo : threadDemos) {
            threadDemo.start();
        }
        // wait other thread complete
        for (ThreadDemo threadDemo : threadDemos) {
            threadDemo.join();
        }
        System.out.println("completing");
    }
}
```

#### 通过实现Callable创建线程(可以对线程返回值处理)

通过`FutureTask`包装一个`Callable`的实例，再通过`Thread`包装`FutureTask`的实例，然后调用`Thread`的`start()`方法

```java
public class CallableDemo implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "yo!";
    }

    @Test
    public void callUse() throws Exception {
        CallableDemo callableDemo = new CallableDemo();
        System.out.println(callableDemo.call());
    }

    @Test
    public void threadUse() throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask= new FutureTask<>(new CallableDemo());
        Thread thread=new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
    }
}
```

FutureTask继承关系

<center>

<img src="https://cdn.guohezuzi.cn/public/img/futureTask.png" width="70%" />

</center>


## 线程池执行线程

#### 线程池的创建

一般通过ThreadPoolExecutor类来创建线程

```java
ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(
                          int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler
                          )
```

##### 变量说明

- corePoolSize 线程池的基本大小

- maximumPoolSize 线程池的最大大小

- keepAliveTime 空闲线程(超出基本大小的线程)的存活时间

- unit 空闲线程存活时间的单位(毫秒,秒...)

- workQueue 任务队列，提交的任务的阻塞队列(`BlockingQueue`)。more: [Java多线程 - 阻塞队列详解](https://www.guohezuzi.cn/article/java-multhread-blockingqueue)

- threadFactory 线程工产，线程的创建策略，有默认实现，可以通过定制线程工厂来监听线程信息 

- handler 饱和策略，当线程由于任务队列满了，或者某个任务被提交到一个已被关闭的线程的处理方式

  - AbortPolicy 中止策略，**默认策略** ，该策略会抛出RejectExecutionException异常，调用者可以根据这个异常编写自己的处理代码

  - DiscardRunsPolicy 抛弃策略，悄悄抛弃该任务，不抛异常

  - DiscardOldestPolicy 抛弃最久任务策略 将工作队列中最老的（也就是下一个要执行的）任务抛弃。**优先队列将会是优先级最高的**

  - CallerRunsPolicy 调用者执行策略，将线程添加到添加工作队列的线程去执行

ps: 构造器参考下表

##### 继承关系

<center>

<img src="https://cdn.guohezuzi.cn/public/img/thread.png" width="70%" />

</center>

#### 线程池的使用

##### Runable接口

```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService pool=Executors.newFixedThreadPool(2);
        pool.execute(() -> System.out.println("yo!"));
        pool.shutdown();
}
```

##### Callable接口

通过调用submit方法

在`ExecutorService`中提供了重载的`submit()`方法，该方法既可以接收`Runnable`实例又能接收`Callable`实例。对于实现`Callable`接口的类，需要覆写`call()`方法，并且只能通过`ExecutorService`的`submit()`方法来启动`call()`方法

```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService pool=Executors.newFixedThreadPool(2);
        Future future=pool.submit(() -> {
            Thread.sleep(100);
            return "yo!";
        });
        System.out.println(future.get());
        pool.shutdown();
}
```

##### 延时任务与周期任务的使用

定义：延时任务("在100ms后执行的任务") 周期任务("每10ms执行一次的任务")

使用：通过new ScheduledThreadPoolExector()对象

<center>

<img src="https://cdn.guohezuzi.cn/public/img/ScheduleExecutor.png" width="80%" />

</center>

Demo:

```java
public class ScheduleExecutorDemo implements Runnable {
    private String name;

    public ScheduleExecutorDemo(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(name + " 运行");
    }

    public static void main(String[] args) throws InterruptedException {
        ScheduledExecutorService executorService1 = Executors.newScheduledThreadPool(2);
        // after 10s run
        executorService1.schedule(new ScheduleExecutorDemo("task1"), 10, TimeUnit.SECONDS);
        executorService1.shutdown();

        ScheduledExecutorService executorService2 = Executors.newScheduledThreadPool(2);
        // run per 1s
        executorService2.scheduleAtFixedRate(new ScheduleExecutorDemo("task1"), 
                0, 1, TimeUnit.SECONDS);
        // run per 2s
        executorService2.scheduleWithFixedDelay(new ScheduleExecutorDemo("task2"), 
                0, 2, TimeUnit.SECONDS);
    }
}
```

##### 使用tips

来源：[阿里巴巴Java开发手册](https://github.com/alibaba/p3c)

- 线程资源必须通过线程池提供,不允许在应用中自行显式创建线程。

  说明: 使用线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销,解决资源不足的问题。如果不使用线程池,有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。

- 线程池不允许使用 Executors 去创建,而是通过 ThreadPoolExecutor 的方式,这样的处理方式让写的同学更加明确线程池运行规则,规避资源耗尽的风险。

  说明: Executors 返回的线程池对象的弊端如下:

  - FixedThreadPool 和 SingleThreadPool :允许的请求队列长度为 Integer.MAX_VALUE ,可能会堆积大量的请求,从而导致 OOM 。

  - CachedThreadPool 和 ScheduledThreadPool :允许的创建线程数量为 Integer.MAX_VALUE ,可能会创建大量的线程,从而导致 OOM 。

- 多线程并行处理定时任务时, Timer 运行多个 TimeTask 时,只要其中之一没有捕获抛出的异常,其它任务便会自动终止运行,使用 ScheduledExecutorService 则没有这个问题。

## Ref:

1. JDK1.8.0 源码

2. [Java多线程之Callable接口及线程池](http://codepub.cn/2016/02/01/Java-multi-thread-Callable-interface-and-thread-pool/)

3. [《Java并发编程实战》](https://book.douban.com/subject/10484692/)

4. [《Java多线程编程实战指南(核心篇)》](https://book.douban.com/subject/27034721/)
