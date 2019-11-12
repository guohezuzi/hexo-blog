# Java多线程 - ThreadLocal详解

## 简介

ThreadLocal是一个为线程提供线程本地变量的工具类。它的思想也十分简单，就是**为线程提供一个线程私有的变量副本**，这样多个线程都可以随意更改自己线程的变量，不会影响到其他线程。

## 代码实现

### 整体实现

Threadlocal通过一个内部类ThreadLocalMap实现对不同线程中不同值的映射，key为ThreadLocal，value为Object即要保存的对象。其ThreadLocalMap中值的存储，和java中其他map类似，通过定义一个内部类Entry来实现。不过，这里的Entry时一个WeakReference对象，为了防止内存泄露，一旦线程结束，key:Threadlocal对象就变为一个不可达的对象，这个Entry就可以被GC了。

```java
   public class ThreadLocal<T> {
        ...省略一些成员变量和方法
        static class ThreadLocalMap {
        ...省略一些成员变量和方法
            static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
            ...省略一些成员变量和方法
        }
     }    

}
```

### Get方法

```java
/**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }    
```

在这里实现的get方法逻辑还是比较简单的，其中ThreadLocalMap中的get方法和hashmap类似。就是有一点需要注意，`ThreadLocalMap map = getMap(t);`时，是通过在每个Thread中有一个`ThreadLocal.ThreadLocalMap threadLocals = null;`的成员变量，在set时，我们通过该线程的这个成员变量，找到对应的ThreadLocalMap。

### Set方法

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

这里的put方法也和hashmap中类似，注意点看上面get方法的说明就好了。

ThreadLocalMap中的set方法:

```java
   /**
         * Set the value associated with key.
         *
         * @param key the thread local object
         * @param value the value to be set
         */
        private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```

   其中与hashmap相同都是将map的length设置为2^n，计算hash时，通过`key.threadLocalHashCode & (len-1)`代替取模运算。不同的时，在rehash时threadlocal使用的是**线性探测法**，而hashmap使用的是**链地址法**

```java
/**
* Increment i modulo len.
*/
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```

### Remove方法

```java
public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```

  threadlocalmap中的remove:

```java
/**
 * Remove the entry for key.
 */
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

其中主要是会在将需要移除的元素置为空后，会调用`expungeStaleEntry(i);`方法，将需要移除的元素key、value清除之后，对当前位置当做 staleSlot 并调用 `expungeStaleEntry` 方法进行整理 (rehashing) 的操作，清除key为null的Entry。

```java
/**
 * Expunge a stale entry by rehashing any possibly colliding entries
 * lying between staleSlot and the next null slot.  This also expunges
 * any other stale entries encountered before the trailing null.  See
 * Knuth, Section 6.4
 *
 * @param staleSlot index of slot known to have null key
 * @return the index of the next null slot after staleSlot
 * (all between staleSlot and this slot will have been checked
 * for expunging).
 */
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;
    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;
                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

#### 拓展-调用remove方法的必要性

总结一下什么时候无用的 Entry 会被清理：

- Thread 结束的时候
- 插入元素时，发现 staled entry，则会进行替换并清理
- 插入元素时，`ThreadLocalMap`  的  `size`  达到  `threshold`，并且没有任何 staled entries 的时候，会调用  `rehash`  方法清理并扩容
- 调用  `ThreadLocalMap`  的  `remove`  方法或`set(null)`  时

尽管不会造成内存泄露，但是可以看到无用的 Entry 只会在以上四种情况下才会被清理，这就可能导致一些 Entry 虽然无用但还占内存的情况。因此，我们在使用完 ThreadLocal 后一定要`remove`一下，保证及时回收掉无用的 Entry。

特别地，当应用线程池的时候，由于线程池的线程一般会复用，Thread 不结束，这时候用完更需要  `remove`  了。

## 具体使用

参考:[ Java多线程 - Spring中的线程安全](https://www.guohezuzi.cn/article/java-multithread-spring)

## Ref

[ 并发编程 | ThreadLocal 源码深入分析](https://www.sczyh30.com/posts/Java/java-concurrent-threadlocal/)
