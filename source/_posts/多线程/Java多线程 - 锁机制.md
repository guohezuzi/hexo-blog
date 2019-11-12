# Java多线程 - 锁机制

## 锁的作用

在不同线程中，对同一变量、方法或代码块进行同步访问

## 锁的实现方式

我们通过一个例子了解锁的不同实现，开启100个线程对同一`int`变量进行`++`操作1000次，在这个过程中如何对这个变量进行同步

未同步代码：

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * \* Created with IntelliJ IDEA.
 * \* User: guohezuzi
 * \* Date: 2018-04-30
 * \* Time: 上午11:26
 * \* Description:自己编写的多线程的栗子(多个线程添加元素到数组中)
 * \
 *
 * @author guohezuzi
 */
public class MyExample {
    private int count = 0;

    class addHundredNum extends Thread {
        @Override
        public void run() {
            //...执行其他操作
            for (int i = 0; i < 1000; i++) {
                    count++;
            }
            //...执行其他操作
        }
    }

    public void test() throws InterruptedException {
        addHundredNum[] addHundredNums = new addHundredNum[100];
        for (int i = 0; i < addHundredNums.length; i++) {
            addHundredNums[i] = new addHundredNum();
        }

        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.start();
        }
        // 等待所有addHundredNum线程执行完毕
        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.join();
        }
    }

    public static void main(String[] args) throws Exception {
        MyExample example = new MyExample();
        example.test();
        System.out.println(example.count);
    }
}
```

### synchronized

通过`synchronized(addHundredNum.class)`给当前对象加锁**而不是**`synchronized(this)`给对象实例加锁

```java
public class MyExample {
    private int count = 0;

    class addHundredNum extends Thread {
        @Override
        public void run() {
            //...执行其他操作
            synchronized (addHundredNum.class) {
            for (int i = 0; i < 1000; i++) {
                    count++;
            }
            }
            //...执行其他操作
        }
    }

    public void test() throws InterruptedException {
        addHundredNum[] addHundredNums = new addHundredNum[100];
        for (int i = 0; i < addHundredNums.length; i++) {
            addHundredNums[i] = new addHundredNum();
        }

        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.start();
        }

        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.join();
        }
    }

    public static void main(String[] args) throws Exception {
        MyExample example = new MyExample();
        example.test();
        System.out.println(example.count);
    }
}
```

#### 拓展

##### synchronized的不同加锁方式

- 给对象加锁
  - 修饰静态方法
  - 修饰代码块时使用synchronized(class)
- 给对象实例加锁
  - 修饰非静态方法
  - 修饰代码块时使用synchronized(this) 或 synchronized(Object)

##### JVM角度理解synchronized关键字

   synchronized关键字经过编译之后，会在同步块的前后分别形成monitorenter和monitorexit这两个字节码指令，这两个字节码都需要一个reference类型的参数来指明要锁定和解锁的对象。如果Java程序中的synchronized明确指定了对象参数，那就是这个对象的reference；如果没有明确指定，那就根据synchronized修饰的是实例方法还是类方法，去取对应的对象实例或Class对象来作为锁对象。
   根据虚拟机规范的要求，在执行monitorenter指令时，首先要去尝试获取对象的锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1，相应地，在执行monitorexit指令时会将锁计数器减1，当计数器为0时，锁就被释放了。如果获取对象锁失败了，那当前线程就要阻塞等待，直到对象锁被另外一个线程释放为止。

##### JDK1.6后对sysnchronized的优化

在JDK1.6之前，使用sysnchronized同步时，如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高

JDK1.6之后，JVM对sysnchronized进行了大量优化，从原来的重量级锁到现在的锁的不同阶段升级 无锁 -> 偏向锁 -> 轻量级锁及自旋锁 -> 重量级锁

- 偏向锁 
  
    当进行同步时，偏向于第一个获得它的线程，如果在接下来的执行中，该锁没有被其他线程获取，那么持有偏向锁的线程就不需要进行同步
  
    但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，此时，偏向锁会升级为轻量级锁

- 轻量级锁
  
    是指当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过CAS自旋的形式尝试获取锁，不会阻塞，从而提高性能。
  
    但如果存在锁竞争，除了互斥量开销外，还会额外发生CAS操作，因此在有锁竞争的情况下，轻量级锁比传统的重量级锁更慢！如果锁竞争激烈，那么轻量级将很快膨胀为重量级锁！

- 自旋锁和自适应自旋锁
  
    轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。让线程自旋的方式等待一段时间
  
    自适应的自旋锁：自旋的时间不在固定了，而是和前一次同一个锁上的自旋时间以及锁的拥有者的状态来决定。

- 锁消除
  
    指的就是虚拟机即使编译器在运行时，如果检测到那些共享数据不可能存在竞争，那么就执行锁消除。锁消除可以节省毫无意义的请求锁的时间。

- 锁粗化
  
    如果一系列的连续操作都对同一个对象反复加锁和解锁，会带来很多不必要的性能消耗，通过对连续操作的一次加锁和解锁(及锁的粗化)来节省时间

### 显式锁

通过JDK层面AQS实现的锁，需要我们通过编程实现，如调用lock()、unlock()

```java
public class MyExample {

    private int count = 0;
    private final Lock lock = new ReentrantLock();

    class addHundredNum extends Thread {
        @Override
        public void run() {
            lock.lock();
            try {
                for (int i = 0; i < 1000; i++) {
                    count++;
                }
            } finally {
                lock.unlock();
            }
        }
    }

    public void test() throws InterruptedException {
        addHundredNum[] addHundredNums = new addHundredNum[100];
        for (int i = 0; i < addHundredNums.length; i++) {
            addHundredNums[i] = new addHundredNum();
        }

        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.start();
        }

        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.join();
        }
    }

    public static void main(String[] args) throws Exception {
        MyExample example = new MyExample();
        example.test();
        System.out.println(example.count);
    }
}
```

AQS详解参考:[JAVA多线程 - AQS详解](https://www.guohezuzi.cn/article/java-multithread-aqs)

### CAS操作

通过使用原子类的CAS方法来实现

```java
public class MyExample {
    private AtomicInteger count = new AtomicInteger(0);

    class addHundredNum extends Thread {
        @Override
        public void run() {
                for (int i = 0; i < 1000; i++) {
                    count.getAndAdd(1);
                }
        }
    }

    public void test() throws InterruptedException {
        addHundredNum[] addHundredNums = new addHundredNum[100];
        for (int i = 0; i < addHundredNums.length; i++) {
            addHundredNums[i] = new addHundredNum();
        }

        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.start();
        }

        for (addHundredNum addHundredNum : addHundredNums) {
            addHundredNum.join();
        }
    }

    public static void main(String[] args) throws Exception {
        MyExample example = new MyExample();
        example.test();
        System.out.println(example.count);
    }
}
```

JDK8可以使用新增LongAdder类实现，该类本身会分成多个区域，多线程写入时，写入对应区域，读取会将整个区域统计输入。

#### 拓展

##### 什么是CAS操作

CAS全称 Compare And Swap（比较与交换），是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。

CAS算法涉及到三个操作数：

- 需要读写的内存值 V。
- 进行比较的值 A。
- 要写入的新值 B。

当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

##### CAS操作存在的问题

1. **ABA问题**。CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。
   - JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。
2. **循环时间长开销大**。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。
3. **只能保证一个共享变量的原子操作**。对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。
   - Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

### volatile

volatile关键字使用时，只能作用于变量，且并不能保证不同线程中的同步，故无法实现上面的同步的例子，接下来我们来介绍一下`volatile`关键字的作用:

1. 保证不同线程中变量的可见性
   
   volatile英译易挥发的，表示修饰的变量是不稳定的，易改变，故采用volatile修饰后，会将变量放到主内存中，不会放到每个线程的cpu高速缓存后在读取，而是直接所用线程都通过到主内存去读取，以保证变量在每个线程的可见性。
   
   然而，这并不意味着变量的线程安全，不同线程cpu进行运算存在时间差，如当多个线程同时对该变量进行`++`操作时，可能其中一个线程读取时变量值为1，这时另外一个线程也读取变量值为1，第一个线程cpu进行`+1`操作运行完毕并已经写回内存，而另一个线程cpu才进行+1操作运算并写入内存，此时一个线程的结果被覆盖，导致线程不安全。

2. 防止新建对象的重排序现象
   
    当变量采用volatile修饰后，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。如保守策略的JMM内存屏障插入策略：
- 在每个volatile写操作的前面插入一个StoreStore屏障。

- 在每个volatile写操作的后面插入一个StoreLoad屏障。

- 在每个volatile读操作的后面插入一个LoadLoad屏障。

- 在每个volatile读操作的后面插入一个LoadStore屏障。

具体例子可参考文章[双重校验锁实现的单例模式](https://www.guohezuzi.cn/article/java-multithead-singleton)中的volatile关键字的作用

## Ref

1. 《深入理解Java虚拟机：JVM高级特性与最佳实践》第十三章

2. [不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)

3. [Java并发：volatile内存可见性和指令重排](http://www.importnew.com/23535.html)
