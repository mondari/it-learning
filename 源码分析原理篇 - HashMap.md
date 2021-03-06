[TOC]



## 数据结构（数组和链表的优缺点）

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



### 知识点：元素在HashMap中的位置怎么决定？为什么HashMap的容量一定要是2的幂？

因为元素在HashMap中存放的位置是由公式 `HashCode%Capacity` 决定的，HashCode是该元素的键的哈希值，Capacity是HashMap的容量。

如果HashMap容量是2的幂，则可以将上述公式优化为计算机更擅长的位运算公式 `HashCode&(Capacity-1)`，两者计算结果是一样的，但是后者速度更快。

## HashMap 中有哪些重要的方法？

### resize 方法

初始化容量和扩容

### tableSizeFor 方法

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



### hash 方法

HashMap 中 key 的哈希值并不是通过 hashCode 方法获得的，而是通过 HashMap 中的 hash 方法获得的。如以下代码所示

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

由上可知，HashMap 中 key 的哈希值其实是通过 `key.hashCode() ^ (h >>> 16)` 来计算的，`h >>> 16` 是将哈希值无符号右移16位，高位补零。然后和自身进行与运算得到哈希值。这样子做就是为了让低16位同时包含高位和低位的信息，在计算下标时，由于高位和低位的同时参与，可以减少哈希碰撞。

## 扩容机制



### 什么是负载因子？负载因子太小或太大会怎样？

什么是负载因子？负载因子表示 HashMap 容量的多少作为负载来存放元素。负载因子 = 元素个数 / 容量。

负载因子默认是 0.75f（DEFAULT_LOAD_FACTOR），也就是3/4，适合大多数场景，一般不需要更改，因为太大或太小会有以下问题：

- **负载因子越小，空间利用率越低，扩容越频繁；**

- **负载因子越大，哈希碰撞的概率就越大，更多冲突的元素存到链表中，查找成本提高**。

**负载因子是可以大于1的**，这样就会导致阈值大于容量（阈值=容量*负载因子），哈希碰撞的概率上升，更多数据会被存放在链表里，进而导致查找成本提高。这就好比将20个数据存放在大小为16的数组里，数组肯定是放不下的，所以多出的数据就会存放在链表里。

### 扩容的条件？如何扩容？

扩容的条件：当 HashMap 的大小超过阈值时，就会发生扩容。阈值 = 容量 * 负载因子。也就是说当 HashMap 的大小超过容量的 3/4 时，就会扩容。

扩容主要有两个操作，**扩容（容量变为原来的两倍）**和**重新哈希（重新分布元素的位置）**。

扩容代码的主要实现在 resize 方法中：

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
        // 须保证扩容后的容量不超过最大容量
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

由上可知，扩容的基本步骤是在保证扩容后的容量不超过 HashMap 的最大容量的情况下，容量变为原来的两倍。

## 红黑树的抽象数据类型

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
      
    //...
}
```

## 其它

### 树化和反树化的条件？

- 树化的条件：链表的长度达到树化阈值8 (TREEIFY_THRESHOLD)，**并且**哈希表的容量达到最小树化容量64 (MIN_TREEIFY_CAPACITY)，才会将链表转为红黑树。

- 反树化的条件：红黑树的节点数达到反树化阈值6 (UNTREEIFY_THRESHOLD)。树化阈值和反树化阈值两者之间隔了个1的原因是避免**链表和红黑树频繁转换**。



树化的源码：

```java
for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            treeifyBin(tab, hash);
        break;
    }
	//...
}

// 其中 treeifyBin 方法如下：
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)// 需要哈希数组的长度大于等于64才树化
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        //...
    }
}
```



反树化的源码：

```java
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    //...
    
	if (loHead != null) {
        if (lc <= UNTREEIFY_THRESHOLD)
            tab[index] = loHead.untreeify(map);
        else {
            tab[index] = loHead;
            if (hiHead != null) // (else is already treeified)
                loHead.treeify(tab);
        }
    }
    if (hiHead != null) {
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            tab[index + bit] = hiHead;
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```



参考：[JDK1.8以后的hashmap为什么在链表长度为8的时候变为红黑树](https://blog.csdn.net/baidu_37147070/article/details/98785367)



### *为什么树化阈值是8？

源码中注释说过这个问题，根据统计，链表中节点数是8的概率差不多是千万分之六，所以一般情况下，链表长度是很难达到8。即使达到8，也要满足哈希表的容量为64，才能树化。这样设计的目的是不到玩不得已的情况下，才让链表树化。

参考：[JDK1.8以后的hashmap为什么在链表长度为8的时候变为红黑树](https://blog.csdn.net/baidu_37147070/article/details/98785367)



### 为什么 HashMap 不直接使用红黑树而是链表？

链表的时间复杂度是O(n)，红黑树的时间复杂度O(logn)，很显然，红黑树的时间复杂度是优于链表，但为什么不直接用红黑树呢？

源码中的注释写的很清楚，**因为红黑树的节点所占的空间是链表节点的两倍，所以只有当链表节点足够多以至于影响查询性能时，才会使用树节点**

> ```
> *
> * Because TreeNodes are about twice the size of regular nodes, we
> * use them only when bins contain enough nodes to warrant use
> * (see TREEIFY_THRESHOLD). And when they become too small (due to
> * removal or resizing) they are converted back to plain bins.  In
> * usages with well-distributed user hashCodes, tree bins are
> * rarely used.  Ideally, under random hashCodes, the frequency of
> * nodes in bins follows a Poisson distribution
> * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
> * parameter of about 0.5 on average for the default resizing
> * threshold of 0.75, although with a large variance because of
> * resizing granularity. Ignoring variance, the expected
> * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
> * factorial(k)). The first values are:
> ```

参考：[JDK1.8以后的hashmap为什么在链表长度为8的时候变为红黑树](https://blog.csdn.net/baidu_37147070/article/details/98785367)

### HashMap 高并发下存在的问题？

HashMap 在扩容的时候会调用 resize 方法，resize 方法主要是负责创建新的哈希表，并将旧的哈希表上的元素重新哈希到新的哈希表上。但是在重新哈希时多个线程同时操作可能会导致链表出现环，如果查找一个不存在 value 的 key，而这个 key 的下标刚好是链表出现环的单元上，则会导致程序出现死循环。

参考：https://mp.weixin.qq.com/s/dzNq50zBQ4iDrOAhM4a70A

### *哈希函数的构造方法有哪些？



### *解决哈希冲突的方法有哪些？

- **链地址法|拉链法**：**HashMap 采用的方法**。将哈希值冲突的元素组成一个同义词单链表，链表的头指针存在哈希表哈希冲突的单元上。

  **链地址法适用于经常插入和删除的情况。**

  缺点：需要额外的存储空间

- **开放定址法**：从发生哈希冲突的那个单元起，按照一定的顺序，从哈希表中找到一个空闲的单元来存放冲突的元素。开放定址法需要的表长度要大于等于所需要存放的元素。

  - **线性探查法(Linear Probing)**：线行探查法是开放定址法中最简单的冲突处理方法，它从发生冲突的单元起，依次探查下一个单元是否为空，当达到最后一个单元时，再从表首依次判断。直到碰到空闲的单元或者探查完全部单元为止。

    如何探查完全部单元都不为空，怎么办？

  - **平方探查法(Quadratic Probing)**：发生冲突时，用发生冲突的单元d<sub>[i]</sub>，加上 1²、 2²等，即 d<sub>[i]</sub> + 1²，d<sub>[i]</sub> + 2², d<sub>[i]</sub> + 3²...，**直到找到空闲单元**。
    在实际操作中，平方探查法不能探查到全部剩余的单元。不过在实际应用中，能探查到一半单元也就可以了。若探查到一半单元仍找不到一个空闲单元，表明此散列表太满，应该重新建立。
    
  - **双散列函数探查法**：

  - **随机探测法**：发生冲突时，随机找个空闲的单元存放冲突的元素。

  缺点：冲突的元素容易堆积在一起

- **再哈希法**：就是同时构造多个不同的哈希函数，当其中一个哈希函数发生冲突时，就用另一个哈希函数进行运算，直到不再产生哈希冲突为止。

  缺点：这种方法不易产生聚集，但是**增加了计算时间**

- **建立公共溢出区**：将哈希表分为公共表和溢出表，发生冲突的元素都放入溢出表中。

  缺点：查找冲突的元素时，需要遍历溢出表

参考：

1. [解决哈希冲突的常用方法分析](https://www.jianshu.com/p/4d3cb99d7580)
2. [哈希表（HashTable）的构造方法和冲突解决](https://www.jianshu.com/p/7e7f52a49ffc)
3. [构造hash函数的方法、解决冲突的方法、常见hash算法](https://www.jianshu.com/p/436175785dfb)

### *常见的哈希算法有哪些？

- MD2：MD 是 Message Digest Algorithm 的简称，中文名为消息摘要算法
- MD4
- MD5：加密后的长度是128位
- SHA-1：SHA 是 Secure Hash Algorithm 的简称，中文名为安全哈希|散列算法。SHA-1加密后长度是160位
- SHA-2（SHA-256、SHA-384、SHA-512）

## 总结

### HashMap Put一个元素的流程

1. 如果是第一次插入元素，会调用 resize 方法初始化容量，默认容量是16，如果在 `new HashMap()` 的时候指定了容量，那么 HashMap 会取大于等于指定容量的2的指数次方作为容量。比如如果指定容量是15，则 HashMap 会取16作为容量。
2. 调用 hash 方法计算 key 的哈希值，并计算下标作为插入位置
3. 判断插入位置是否为空，是则直接插入，否则走下一步
4. 判断已有元素和插入元素的哈希值和键值是否相等，相等则覆盖，不相等则走下一步
5. 判断已有元素是链表节点还是树节点，然后遍历链表或树，查找有无 key 相同的节点，有则覆盖，没有则新增
6. 判断是否要树化和反树化
7. 判断是否要扩容

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

