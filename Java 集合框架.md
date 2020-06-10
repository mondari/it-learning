[TOC]

## List


- ArrayList：有序，可重复，支持空元素，线程不安全，初始容量为10。
- Vector：有序，可重复，支持空元素，线程安全，初始容量为10。Vector 其实就是异步的ArrayList，其它都和 Vector 相同。
- LinkedList：双向链表，有序，可重复，支持空元素，线程不安全。
- Stack：栈，Vector 的子类，一般不推荐使用，而是使用 Deque 接口的实现类。



- Arrays工具类里的 sort 方法使用了双轴快速排序算法（Dual-Pivot QuickSort），binarySearch 方法使用了二分查找算法。

## 简单介绍一下 ArrayList？

ArrayList 的内部结构是数组（Object 数组），数组的大小会根据元素的个数动态增长。数组的初始大小是10。每次添加元素，都会判断元素的个数是否会超过数组的大小，如果超过就会发生扩容，扩容后的容量变为原来的1.5倍。扩容的主要操作是通过 `Arrays.copyOf(elementData, newCapacity)` 将旧的数组复制到容量更大的新数组。

## ArrayList 和 数组的区别

- ArrayList：基于动态数组，容量可以动态增长，但由于扩容时使用 Arrays.copyOf 复制数组，所以会牺牲一定的性能；
- 数组：由于容量固定无法动态增长，所以插入性能比 ArrayList 高；

总结：优先使用数组，无法确定数组大小时才使用 ArrayList ！

## Map

- HashMap：无序，支持空的键和值，非线程安全，初始容量设置过高会影响遍历性能。默认初始容量为16，默认负载系数为0.75。在哈希散列正常的情况下，可以提供常数时间复杂度的添加和删除操作。
- Hashtable：无序，**不支持空的键和值，线程安全**，初始容量设置过高会影响遍历性能。默认初始容量为11，默认负载系数为0.75。
- LinkedHashMap：HashMap 的子类，双向链表实现，有序（按插入顺序或访问顺序排序，按访问顺序排序是指每次调用 get 方法访问元素时，会将访问的元素挪到链表的尾部），键和值允许为空，非线程安全，**容量大小不影响遍历性能。**
- TreeMap：红黑树实现，有序（按自然顺序或指定顺序排序，按指定顺序排序需要实现 Comparator 接口），**键不能为空，但值允许为空，**非线程安全。添加、删除操作的时间复杂度是log(n)。

PS：LinkedHashMap 可以自定义删除策略

```java
LinkedHashMap<String, String> accessOrderedMap = new LinkedHashMap<String, String>(16, 0.75F, true){
    @Override
    protected boolean removeEldestEntry(Map.Entry<String, String> eldest) { // 自定义删除策略，元素超过3个则自动删除最早的元素
        return size() > 3;
    }
};
```



## 简单介绍一下 HashMap？

HashMap 其内部数据结构是数组+链表，默认容量是16，

## HashMap 和 Hashtable 的区别？

两者都是哈希表实现，其中 Hashtable 是线程安全版本的 HashMap，不支持空的键和值，而 HashMap 支持。

## LinkedHashMap 和 TreeMap 的区别？

两者都是有序的哈希表实现，其中 LinkedHashMap 是 HashMap 的子类，它是按照插入顺序或访问顺序排序，而 TreeMap 是按照自然顺序或指定顺序排序，且键不能为空，但值允许为空。

## LinkedHashMap 的实现

LinkedHashMap 重新定义了 HashMap 的 HashMap.Node，增加上一个元素 before 和下一个元素 after 的引用，从而在哈希表的基础上构建一个双向链表。

```java
/**
 * HashMap.Node subclass for normal LinkedHashMap entries.
 */
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

参考：[LinkedHashMap 的实现原理](http://wiki.jikexueyuan.com/project/java-collection/linkedhashmap.html)



## Set

- HashSet：无序，不可重复，支持空元素，线程不安全，初始容量为16，默认负载系数为0.75。初始容量设置过高会影响遍历性能。HashSet的底层实现是HashMap，HashMap的key就是HashSet的值，HashMap的value是固定的Object，这样达到了HashSet的值不重复的效果。
- LinkedHashSet：双向链表，有序（按插入顺序排序），不可重复，支持空元素，线程不安全，容量大小不影响遍历性能。
- TreeSet：有序，不可重复，**元素不能为空**，线程不安全

## Queue 和 Deque 和 Stack

- Queue：队列，FIFO 的数据结构
- Deque：支持对头尾都进行插入和删除的队列
- Stack：栈，LIFO 的数据结构



Queue 提供以下操作方法：

|             | *Throws exception*                                           | *Returns special value*                                      |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#offer-E-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#remove--) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#poll--) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) |

Deque 提供以下操作方法：

|             | **First Element (Head)**                                     | **Last Element (Tail)**                                      |                                                              |                                                              |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|             | *Throws exception*                                           | *Special value*                                              | *Throws exception*                                           | *Special value*                                              |
| **Insert**  | [`addFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addFirst-E-) | [`offerFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerFirst-E-) | [`addLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addLast-E-) | [`offerLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerLast-E-) |
| **Remove**  | [`removeFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeFirst--) | [`pollFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollFirst--) | [`removeLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeLast--) | [`pollLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollLast--) |
| **Examine** | [`getFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getFirst--) | [`peekFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekFirst--) | [`getLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getLast--) | [`peekLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekLast--) |

Queue 和 Deque 操作方法的对比：

| **Queue Method**                                             | **Equivalent Deque Method**                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#add-E-) | [`addLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addLast-E-) |
| [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#offer-E-) | [`offerLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerLast-E-) |
| [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#remove--) | [`removeFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeFirst--) |
| [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#poll--) | [`pollFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollFirst--) |
| [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`getFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getFirst--) |
| [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) | [`peekFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peek--) |

Stack 和 Deque 操作方法的对比：

| **Stack Method**                                             | **Equivalent Deque Method**                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`push(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#push-E-) | [`addFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addFirst-E-) |
| [`pop()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pop--) | [`removeFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeFirst--) |
| [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peek--) | [`peekFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekFirst--) |

## BlockingQueue

BlockingQueue 提供以下操作方法，它比 Queue 接口新增了两列操作 Blocks 和 Times out。


|             | *Throws exception*                                           | *Special value*                                              | *Blocks*                                                     | *Times out*                                                  |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-) | [`put(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#put-E-) | [`offer(e, time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-long-java.util.concurrent.TimeUnit-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#remove-java.lang.Object-) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) | [`take()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#take--) | [`poll(time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) | *not applicable*                                             | *not applicable*                                             |

## 有界队列和无界队列

有界队列：ArrayBlockingQueue（基于数组，界限就是容量）、LinkedBlockingQueue（基于链表，理论上是无界的，但实际上是有界的，如果不指定容量，则默认最大容量是Integer.MAX_VALUE）

无界队列：PriorityBlockingQueue、ConcurrentLinkedQuque

SynchronousQueue：容量为0，每个删除操作都要等待插入操作，反之每个插入操作也都要等待删除动作

## 为什么使用 ConcurrentHashMap？

因为 HashMap 不是线程安全，而 Hashtable 虽然是线程安全的，但也只是简单地把 HashMap 的方法声明为 synchronized 来保证线程安全，这样就会导致所有并发操作都竞争同一把锁，一个线程在同步操作时，另一个线程只能等待，大大降低了并发操作的效率。另外，使用 Collections.synchronizedMap 方法提供的同步包装器功能也无法满足高并发的需求，虽然它的方法不再声明为 synchronized，但是它只是利用传入的 Map 来构造另一个版本，使用 this 作为同步监视器，并没有真正意义上的改进！所以，Hashtable 和 Collections.synchronizedMap 的同步包装器只适合低并发的场景。

```java
private static class SynchronizedMap<K,V>
    implements Map<K,V>, Serializable {
    private static final long serialVersionUID = 1978198479659022715L;

    private final Map<K,V> m;     // Backing Map
    final Object      mutex;        // Object on which to synchronize

    SynchronizedMap(Map<K,V> m) {
        this.m = Objects.requireNonNull(m);
        mutex = this;
    }

    SynchronizedMap(Map<K,V> m, Object mutex) {
        this.m = m;
        this.mutex = mutex;
    }

    public int size() {
        synchronized (mutex) {return m.size();}
    }
    
    //...
}
```

## ConcurrentHashMap 如何实现？

JDK 7 和 JDK 8 两个版本的区别。

JDK 8 中，ConcurrentHashMap 使用 volatile 保证了哈希表的可见性。

参见：[源码分析原理篇 - ConcurrentHashMap.md](源码分析原理篇 - ConcurrentHashMap.md)