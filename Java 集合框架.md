[TOC]


- ArrayList：有序，可重复，支持空元素，线程不安全，初始容量为10。
- Vector：有序，可重复，支持空元素，线程安全，初始容量为10。Vector 其实就是异步的ArrayList，其它都和 Vector 相同。
- LinkedList：双向链表，有序，可重复，支持空元素，线程不安全。
- Stack：栈，继承 Vector，一般不推荐使用，而是使用 Deque 接口的实现类。



- Arrays工具类里的sort方法使用了快速排序算法，binarySearch方法使用了二分查找算法。



HashMap 和 Hashtable 都是哈希表实现，两者大体上相同，不同的是前者是非线程安全，支持空的键和值，而后者是线程安全，不支持空的键和值。

- HashMap：无序，键和值允许为空，非线程安全，初始容量设置过高会影响遍历性能。初始容量为16，默认负载系数为0.75。
- Hashtable：无序，**键和值不允许为空，线程安全**，初始容量设置过高会影响遍历性能。初始容量为11，默认负载系数为0.75。
- LinkedHashMap：有序（按插入顺序排序），key和value允许为空，线程不安全，**容量大小不影响遍历性能。**
- TreeMap：实现SortedMap和NavigableMap接口，有序（按自然顺序排序），**key不可为空，但value允许为空，**非线程安全。红黑树原理。



- HashSet：无序，不可重复，支持空元素，线程不安全，初始容量为16，默认负载系数为0.75。初始容量设置过高会影响遍历性能。HashSet的底层实现是HashMap，HashMap的key就是HashSet的值，HashMap的value是固定的Object，这样达到了HashSet的值不重复的效果。
- LinkedHashSet：双向链表，有序（按插入顺序排序），不可重复，支持空元素，线程不安全，容量大小不影响遍历性能。
- TreeSet：有序，不可重复，**元素不能为空**，线程不安全



- Queue
- Deque



## 为什么使用 ConcurrentHashMap？

因为 HashMap 不是线程安全，而 Hashtable 虽然是线程安全的，但也只是简单地把 HashMap 的方法声明为 synchronized 来保证线程安全，这样就会导致所有并发操作都竞争同一把锁，一个线程在同步操作时，另一个线程只能等待，大大降低了并发操作的效率。另外，使用 Collections 工具类提供的同步包装器 SynchronizedMap 也无法满足高并发的需求，虽然它的方法不再声明为 synchronized，但是它只是利用传入的 Map 来构造另一个版本，使用 this 作为同步监视器，并没有真正意义上的改进！所以，Hashtable 和 Collections 的同步包装器只是适合低并发的场景。

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