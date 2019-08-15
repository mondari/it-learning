# HashMap原理

[TOC]



## 数据结构

HashMap的数据结构是数组和链表，它结合了数组和链表两者的优点，具有查询快，插入快的特点。

数组的优点是查询快，因为它有下标，可以快速定位元素；缺点是插入慢，需要将插入位置后面的元素统统向后挪一位。

链表的优点是插入快，插入元素时只需要修改节点的指向关系即可；缺点是查询慢，需要遍历整个链表去查询。



那么HashMap是怎么结合数组和链表两者的优点的呢？

一般情况下，HashMap使用数组来存放元素，该数组叫桶数组（Buckets），其内部定义是 `Node<K,V>[] table` ，`Node` 其实就是链表节点，其代码如下

```java
/**
 * Basic hash bin node, used for most entries.
 */
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    	//......
}
```

和一般数组不同，元素不是按照插入顺序依次放入到桶数组中，其在桶数组中存放的位置其实是由公式 `HashCode%Capacity` 决定的。HashCode是该元素的键的哈希值，Capacity是桶数组大小，也是HashMap的容量。这样向数组中插入元素就不用挪动后面的元素，巧妙地解决了数组插入慢的缺点。

当存在哈希碰撞时，通过Node.next指向哈希碰撞的元素，从而将哈希碰撞的元素存放在链表里。这样HashMap结合了数组和链表的优点，实现快速插入和查询元素。



我们从HashMap是怎么插入一个元素开始，逐个学习HashMap原理的各个知识点。

## 初始化容量

### 知识点：什么时候会初始化容量？

向HashMap插入元素的第一步是初始化容量。因为在 `new HashMap()` 时并不会初始化容量，只有在第一次调用 `put()` 方法插入元素时才会初始化容量。注意， `put()` 方法实际调用的是 `putVal()` 方法，而 `putVal()` 方法里是调用 `resize()` 方法来初始化容量。实际上 `resize()` 方法不仅仅是用来初始化容量，它还可以用来扩容，下面会提到。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //......
}
```

```java
/*resize() 初始化容量代码分析*/
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;

    // 旧哈希表容量和阈值
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    // 新哈希表容量和阈值
    int newCap, newThr = 0;

    // 因为 oldTab == table == null，所以oldCap == 0，所以条件不成立
    if (oldCap > 0) {
        // 扩容代码...
    }
    // 这里有两种情况，
    // 如果没有设置初始容量，则oldThr == threshold == 0, 条件不成立
    // 如果设置了初始容量，则oldThr(== threshold)其实就是tableSizeFor()方法返回的初始容量，初始容量必定是大于0的，所以条件是成立的。
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 这里也有两种情况，
    // 如果没有设置初始容量，则newThr == (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY)，条件不成立;
    // 如果设置了初始容量，则newThr一定为0，因为上面没有给newThr赋予新值，所以条件成立
    if (newThr == 0) {
        // 计算新阈值。阈值是int型，最大不能超过Integer.MAX_VALUE
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    // 设置阈值
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    // 初始化容量核心代码
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;

    // oldTab == table == null，显然条件不成立
    if (oldTab != null) {
        // 重新哈希代码...
    }

    // 返回新哈希表
    return newTab;
}
```



### 知识点：HashMap的默认初始容量是多少？

HashMap的默认初始容量是16，由常量 `DEFAULT_INITIAL_CAPACITY` 定义。如果在 `new HashMap()` 时不指定HashMap的容量的话，就会使用默认初始容量16作为HashMap的容量。

```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```



### 知识点：HashMap的最大容量是多少？

HashMap的最大容量是1<<30，由常量 `MAXIMUM_CAPACITY` 定义。如果在 `new HashMap()` 时设置的容量超过该值，会自动改为最大容量1<<30。

```java
/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 */
static final int MAXIMUM_CAPACITY = 1 << 30;
```



### 知识点：元素在HashMap中的位置怎么决定？为什么HashMap的容量一定要是2的幂？

因为元素在HashMap中存放的位置是由公式 `HashCode%Capacity` 决定的，HashCode是该元素的键的哈希值，Capacity是HashMap的容量。

如果HashMap容量是2的幂，则可以将上述公式优化为计算机更擅长的位运算公式 `HashCode&(Capacity-1)`，两者计算结果是一样的，但是后者速度更快。



### 知识点：HashMap中的tableSizeFor方法

我们在 `new HashMap()` 时，如果指定的参数初始容量不是2的幂的话，HashMap会调用tableSizeFor方法，找到大于或等于初始容量的2的最小幂作为初始容量。例如，我们传入的初始容量大小为3，而大于或等于3的2的最小幂为4，所以HashMap的容量实际上是4而不是3。同理，如果我们传入0，实际上1；传入6，实际上是8；传入10，实际上是16……

tableSizeFor方法的源码如下

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    // 先给容量减1，后面再加1，防止容量已经是2的幂，再进行以下操作就会变为原来的两倍
    int n = cap - 1;
    // 将二进制数最左边的1后面所有位全改为1
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // 返回结果前加1
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

看不懂上面代码的逻辑没关系，我们先一步一步来。

我们知道，2的二进制表示是0001，而0010、0100、1000等都是2的幂，观察这几个数可以发现这个规律：2的幂其二进制表示都只有一个1，且1后面全是0（或者没有0）。我们也知道，除了0外，任何数的二进制表示都至少有一个1。

由此，我们便有一种简便的算法去找到大于一个数的最小2的幂：如果将一个二进制数的位数最高的1后面所有位全改为1，然后再加1，得到的数肯定就是大于原数的2的幂。比如：00010100，将其最左边的1后面所有位都改为1，则变为00011111，再加1，得到00100000，确实是大于00010100的2的幂。

但这算法只能求出大于原数的2的幂，不能求出大于或等于原数的2的幂。如果一个数本来就是2的幂，得到的结果应该是其本身，而不是原数两倍的2的幂。比如00010000，进行上述操作后会得到00100000，而我们想要的是大于或等于00010000的2的幂，也就是原数本身。

怎么解决这个问题呢？我们先看看2的幂有哪些：

1，2，4，8，16。。。

我们发现，2的幂之间最小相差1。如果在进行上述操作前，先将这个数减1得到一个临时数，再通过上面算法求出大于这个临时数的2的幂，结果要么也是大于原数的2的幂，要么是等于原数的2的幂（当原数也是2的幂时），问题就迎刃而解了，这个算法着实牛逼啊！。

至于怎么将一个二进制位数最高的1后面所有位全改为1，看下面

```shell
假设这个数是32768，其二进制表示形式是1000 0000 0000 0000
1000 0000 0000 0000 >> 1 = 0100 0000 0000 0000
0100 0000 0000 0000 | 1000 0000 0000 0000 = 1100 0000 0000 0000

1100 0000 0000 0000 >> 2 = 0011 0000 0000 0000
0011 0000 0000 0000 | 1100 0000 0000 0000 = 1111 0000 0000 0000 

1111 0000 0000 0000 >> 4 = 0000 1111 0000 0000
0000 1111 0000 0000 | 1111 0000 0000 0000 = 1111 1111 0000 0000

1111 1111 0000 0000 >> 8 = 0000 0000 1111 1111
0000 0000 1111 1111 | 1111 1111 0000 0000 = 1111 1111 1111 1111
到这里是不是就将其二进制位数最高的1后面所有位全改为1了？

再举个例子，
0111 0000 0000 0000 >> 1 = 0011 1000 0000 0000
0011 1000 0000 0000 | 0111 0000 0000 0000 = 0111 1000 0000 0000

0111 1000 0000 0000 >> 2 = 0001 1110 0000 0000
0001 1110 0000 0000 | 0111 1000 0000 0000 = 0111 1110 0000 0000 

0111 1110 0000 0000 >> 4 = 0000 0111 1110 0000
0000 0111 1110 0000 | 0111 1110 0000 0000 = 0111 1111 1110 0000

0111 1111 1110 0000 >> 8 = 0000 0000 0111 1111
0000 0000 0111 1111 | 0111 1111 1110 0000 = 0111 1111 1111 1111
到这里是不是也将其二进制位数最高的1后面所有位全改为1了？
```



### 知识点：HashMap中的hash方法

HashMap中key的哈希值并不是通过hashCode方法获得的，而是通过HashMap中的hash方法获得的。如以下代码所示

```java
/**
 * Computes key.hashCode() and spreads (XORs) higher bits of hash
 * to lower.  Because the table uses power-of-two masking, sets of
 * hashes that vary only in bits above the current mask will
 * always collide. (Among known examples are sets of Float keys
 * holding consecutive whole numbers in small tables.)  So we
 * apply a transform that spreads the impact of higher bits
 * downward. There is a tradeoff between speed, utility, and
 * quality of bit-spreading. Because many common sets of hashes
 * are already reasonably distributed (so don't benefit from
 * spreading), and because we use trees to handle large sets of
 * collisions in bins, we just XOR some shifted bits in the
 * cheapest possible way to reduce systematic lossage, as well as
 * to incorporate impact of the highest bits that would otherwise
 * never be used in index calculations because of table bounds.
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

```

由上可知，HashMap中key的哈希值其实是通过 `key.hashCode() ^ (h >>> 16)` 来计算的，`h >>> 16` 是将哈希值无符号右移16位，高位补零。这样子做就是为了让低16位同时包含高位和低位的信息，在计算下标时，由于高位和低位的同时参与，可以减少哈希碰撞。

## 扩容机制

### 知识点：扩容的条件？

当HashMap的大小超过阈值时，就会发生**扩容（容量变为原来的两倍，阈值也变为原来的两倍）**和**重新哈希（重新分布元素的位置）**。阈值的大小就是HashMap容量和负载因子的乘积。

什么是负载因子呢？它是一个大于0的数，表示HashMap容量的多少作为负载，存放元素。负载因子默认是0.75f，由常量 `DEFAULT_LOAD_FACTOR` 控制。也就是说默认情况下，HashMap容量的3/4会作为负载，当HashMap的大小超过容量的3/4，就会扩容和重新哈希。

```java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

截取JDK8官方文档的一段话让大家深入理解负载因子和容量

> An instance of `HashMap` has two parameters that affect its performance: *initial capacity* and *load factor*. The *capacity* is the number of buckets in the hash table, and the initial capacity is simply the capacity at the time the hash table is created. The *load factor* is a measure of how full the hash table is allowed to get before its capacity is automatically increased. When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is *rehashed* (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.

扩容代码如下

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;

    // 旧哈希表容量和阈值
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    // 新哈希表容量和阈值
    int newCap, newThr = 0;
    
    if (oldCap > 0) {
        // 扩容算法
        // 须保证扩容后的容量不超过最大容量的情况下
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 容量变为原来的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            
            // 阈值变为原来的两倍
            newThr = oldThr << 1; // double threshold
    }
    //......
}
```



### 知识点：负载因子太小会怎样，太大会怎样？

默认负载因子大小（0.75）在时间和空间成本之间取得了良好的平衡，适合大多数场景。因为负载因子越小，空间利用率越低，扩容越频繁；负载因子越大，空间开销虽然降低，但查找成本提高。所以，通常情况下不需要改负载因子的大小。

**PS：负载因子是可以大于1的哦，这样就会导致阈值大于容量（阈值=容量*负载因子），哈希碰撞的概率上升，更多数据会被存放在链表里，进而导致查找成本提高。这就好比将20个数据存放在大小为16的数组里，数组肯定是放不下的，所以多出的数据就会存放在链表里。**



### *知识点：树化和反树化条件？

因为链表的查询性能会随着链表长度而下降，所以当HashMap中链表长度大于等于7（树化阈值-1）时会发生树化，将链表改造成红黑树。树化阈值由常量 `TREEIFY_THRESHOLD` 定义，值为8。

HashMap中红黑树的代码如下：

```java
/**
 * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
 * extends Node) so can be used as extension of either regular or
 * linked node.
 */
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    
    //...
}
```

**PS：在一定情况下，HashMap中的红黑树也会发生反树化。研究中。。。**



## 总结

### Hash Map Put一个元素的流程

1. 如果是第一次插入元素，则调用resize方法初始化容量
2. 判断插入位置是否已有元素，否则直接插入，是则下一步
3. 判断已有元素和插入元素的哈希值和键值是否相等，是则覆盖，否则下一步
4. 判断已有元素是链表节点还是树节点，是树节点则执行树节点的插入操作，不是则下一步
5. 遍历链表节点，查看有无键相同的节点，没有则新增，已有则覆盖
6. 是否树化
7. 是否扩容

核心代码解读如下：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        // 第一次插入元素时，会调用resize方法初始化容量
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 插入位置没有元素，则直接插入该元素
        tab[i] = newNode(hash, key, value, null);
    else {
        // 插入位置存在元素，走这里
        Node<K,V> e; K k;
        // 先看已存在元素和插入元素的键是否相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            // 键相同则把已存在元素拿出来，后面会将新值覆盖旧值
            // 注意此时p和e的引用相同，都是已存在元素的引用
            e = p;
        else if (p instanceof TreeNode)
            // 如果插入位置元素是树节点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 如果插入位置元素是链表节点，则遍历链表节点，查找有无键相同的节点
            // 注意，p(pointer)是指针，e(exist)是现有节点，也可以说是下一个节点
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 链表节点长度达到树化阈值时，将链表树化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // e元素和插入元素的键相同时，直接break，下面会统一将新值覆盖旧值
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 只有已存在元素和插入元素的键相同的情况，该条件才会成立
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                // 这里将已存在元素的值覆盖为新值
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 是否要扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

```



### HashMap Get一个元素的流程

1. 根据键的哈希值和HashMap容量计算数组下标
2. 查看数组小标下第一个节点是否为空，空则返回null，否则下一步
3. 查看第一个节点的键是否相同，相同则返回该节点的元素，不同则下一步
4. 第二个节点是链表节点还是树节点，分别执行各自查找键相同的节点的方法
5. 查找键相同的节点则返回该节点的元素，没有则返回null

核心代码解读如下：

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 1. 先看第一个节点的键是否相同
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 2. 判断第二个节点是链表节点还是树节点，是树节点则调用树节点方法来查找键相同的节点
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // 3. 是链表节点则遍历链表，查找键相同的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

