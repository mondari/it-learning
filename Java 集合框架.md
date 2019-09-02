[TOC]

## List


- ArrayList：有序，可重复，支持空元素，线程不安全，初始容量为10。
- Vector：有序，可重复，支持空元素，线程安全，初始容量为10。Vector 其实就是异步的ArrayList，其它都和 Vector 相同。
- LinkedList：双向链表，有序，可重复，支持空元素，线程不安全。
- Stack：栈，Vector 的子类，一般不推荐使用，而是使用 Deque 接口的实现类。



- Arrays工具类里的 sort 方法使用了双轴快速排序算法（Dual-Pivot QuickSort），binarySearch 方法使用了二分查找算法。

## ArrayList 和 数组的区别



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

## Queue

- Queue
- Deque



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