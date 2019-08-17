# Java Collection Framework

- ArrayList：有序，可重复，支持空元素，线程不安全，初始容量为10
- Vector：有序，可重复，支持空元素，线程安全，初始容量为10。Vector 其实就是异步的ArrayList，其它都和 Vector 相同。
- LinkedList：双向链表，有序，可重复，支持空元素，线程不安全



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