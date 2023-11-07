# 1.8

## 简介

HashMap的数据结构是数组和链表，它结合了数组和链表两者的优点，具有查询快，插入快的特点。

数组的优点是查询快，因为它有下标，可以快速定位元素；缺点是插入慢，需要将插入位置后面的元素统统向后挪一位。

链表的优点是插入快，插入元素时只需要修改节点的指向关系即可；缺点是查询慢，需要遍历整个链表去查询。

## 内部属性

```java
// 数组，存放元素
transient Node<K,V>[] table;

// 元素个数
transient int size;

// Iterator fail-fast 机制
transient int modCount;

// 数组初始化前，该值表示容量，为0表示默认容量16；初始化后，该值为阈值=容量*负载因子
int threshold;

// 负载因子，表示容量的多少作为负载
final float loadFactor;

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    // 指向哈希冲突链表下一个节点
    Node<K,V> next;    
    ...
}
```

## 构造方法

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;

public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    // threshold此时表示容量，tableSizeFor方法确保HashMap容量为2的n次方
    this.threshold = tableSizeFor(initialCapacity);
}
```

由上面的构造方法可见，HashMap对象刚创建时其内部数组并没有初始化，而是延迟到首次调用 put 方法。

## put(K key, V value)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
/**
 * put操作的逻辑主要在putVal中。
 * onlyIfAbsent参数表示只有下标位置不存在key相等的元素才插入。
 * evict参数是给LinkedHashMap用的，表示删除最早的节点
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // 变量tab是内部数组，p是节点指针，n是数组长度，i是元素下标
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    
    if ((tab = table) == null || (n = tab.length) == 0)
        // 插入第一个元素时，先resize()初始化数组
        n = (tab = resize()).length;
    
    // 元素下标 = 哈希 % 数组长度，下面是高阶写法
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 空槽直接插入
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 找key相等的节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            // 如果已经树化，则走树化节点的插入逻辑
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 哈希冲突，则沿着链表找空节点或key相等的节点
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    // 当前节点的下一个节点为空，则直接插入
                    p.next = newNode(hash, key, value, null);
                    // 添加节点后达到要树化的条件
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // key相等的节点，则先跳出循环，由下面的步骤替换旧value
                    break;
                // 继续找下一个节点
                p = e;
            }
        }
        // key相等的节点，替换旧value
        // 由于是替换，并没有增加新元素，所以直接返回
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            
            // Callbacks to allow LinkedHashMap post-actions
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // 能走到这里的，都是增加新元素，所以这里要+1
    ++modCount;
    if (++size > threshold)
        // 添加元素后，大小超过阈值，则扩容
        resize();
    
    // Callbacks to allow LinkedHashMap post-actions
    afterNodeInsertion(evict);
    return null;
}
```

## resize()

```java
/**
 * 负责初始化数组和扩容
 */
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 数组长度>0，表示已初始化，则计算扩容的容量和阈值
    // 数组长度<=0，表示未初始化，则计算初始化的容量和阈值。此时threshold表示初始容量，threshold为0表示初始容量为16
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 扩容的容量和阈值变为2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 初始化的容量和阈值
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 重新哈希
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 遍历旧数组的元素
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    // 没有链表结构，则直接哈希到新数组
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 红黑树结构
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 链表结构
                    
                    // loHead 是低位头节点，loTail 是低位尾节点
                    // hiHead 是高位头结点，hiTail 是高位尾结点
                    // 低位节点放到newTab[j]，高位节点放到newTab[j + oldCap]
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // ?
                        // (e.hash & oldCap) == 0) 区分高低位的原理？
                        // 尾插法
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## tableSizeFor(int cap)

```java
/**
 * 将容量向上取整到2的n次方
 */
static final int tableSizeFor(int cap) {
    // 先给容量减1，后面再加1，防止容量已经是2的n次方，再进行以下操作就会变为原来的两倍
    int n = cap - 1;
    // 将二进制数最左边的1后面所有位全改为1
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // 返回结果前加1，补偿前面的减1操作
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

## hash(Object key)

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

## remove(Object key)

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

## get(Object key)

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 1. 先看键是否相同
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 2. 判断节点是链表节点还是树节点，是树节点则调用树节点方法来查找键相同的节点
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

## TreeNode

```java
/**
 * 红黑树节点
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
    ...
}
/**
 * LinkedHashMap 的节点
 */
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
    ...
}
```

## 元素在HashMap中的位置怎么决定？为什么HashMap的容量一定要是2的幂？

因为元素在HashMap中存放的位置是由公式 `HashCode%Capacity` 决定的，HashCode是该元素的键的哈希值，Capacity是HashMap的容量。

如果HashMap容量是2的幂，则可以将上述公式优化为计算机更擅长的位运算公式 `HashCode&(Capacity-1)`，两者计算结果是一样的，但是后者速度更快。

## 什么是负载因子？负载因子太小或太大会怎样？

什么是负载因子？负载因子表示 HashMap 容量的多少作为负载来存放元素，最大元素个数 = 容量 * 负载因子。

负载因子默认是 0.75f（DEFAULT_LOAD_FACTOR），也就是3/4，适合大多数场景，一般不需要更改，因为太大或太小会有以下问题：

- **负载因子越小，空间利用率越低，扩容越频繁；**

- **负载因子越大，哈希碰撞的概率就越大，更多冲突的元素存到链表中，查找成本提高**。

**负载因子是可以大于1的**，这样就会导致更多数据会被存放在链表里，进而导致查找成本提高。这就好比将20个数据存放在大小为16的数组里，数组肯定是放不下的，所以多出的数据就会存放在链表里。

## 扩容的条件？如何扩容？

扩容的条件：当 HashMap 的大小超过阈值时，就会发生扩容。阈值 = 容量 * 负载因子。也就是说当 HashMap 的大小超过容量的 3/4 时，就会扩容。

扩容主要有两个操作，**扩容（容量变为原来的两倍）**和**重新哈希（重新分布元素的位置）**。

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

### *解决哈希冲突的方法有哪些？

- **链地址法|拉链法（Chaining）**：**HashMap 采用的方法**。将哈希值冲突的元素组成一个同义词单链表，链表的头指针存在哈希表哈希冲突的单元上。

  优点：适用于经常插入和删除的情况

  缺点：需要额外的存储空间

- **开放定址法（Open Addressing）**：从发生哈希冲突的那个单元起，按照一定的顺序，从哈希表中找到一个空闲的单元来存放冲突的元素。开放定址法需要的表长度要大于等于所需要存放的元素。

  - **线性探查法(Linear Probing)**：线行探查法是开放定址法中最简单的冲突处理方法，它从发生冲突的单元起，依次探查下一个单元是否为空，当达到最后一个单元时，再从表首依次判断。直到碰到空闲的单元或者探查完全部单元为止。

    如何探查完全部单元都不为空，怎么办？

    源码示例：ThreadLocalMap

  - **平方探查法(Quadratic Probing)**：发生冲突时，用发生冲突的单元d<sub>[i]</sub>，加上 1²、 2²等，即 d<sub>[i]</sub> + 1²，d<sub>[i]</sub> + 2², d<sub>[i]</sub> + 3²...，**直到找到空闲单元**。
    在实际操作中，平方探查法不能探查到全部剩余的单元。不过在实际应用中，能探查到一半单元也就可以了。若探查到一半单元仍找不到一个空闲单元，表明此散列表太满，应该重新建立。
    
  - **随机探测法**：发生冲突时，随机找个空闲的单元存放冲突的元素。

  - **双散列函数探查法**

  缺点：冲突的元素容易堆积在一起

- **再哈希法（Rehashing）**：就是同时构造多个不同的哈希函数，当其中一个哈希函数发生冲突时，就用另一个哈希函数进行运算，直到不再产生哈希冲突为止。

  缺点：这种方法不易产生聚集，但是**增加了计算时间**

- **建立公共溢出区**：将哈希表分为公共表和溢出表，发生冲突的元素都放入溢出表中。

  缺点：查找冲突的元素时，需要遍历溢出表

参考：

1. [解决哈希冲突的常用方法分析](https://www.jianshu.com/p/4d3cb99d7580)
2. [哈希表（HashTable）的构造方法和冲突解决](https://www.jianshu.com/p/7e7f52a49ffc)
3. [构造hash函数的方法、解决冲突的方法、常见hash算法](https://www.jianshu.com/p/436175785dfb)

## 总结

### Put一个元素的流程

1. 如果是第一次插入元素，会调用 resize 方法初始化容量，默认容量是16，如果在 `new HashMap()` 的时候指定了容量，那么 HashMap 会取大于等于指定容量的2的n次方作为容量。比如如果指定容量是15，则 HashMap 会取16作为容量。
2. 调用 hash 方法计算 key 的哈希值，并计算下标作为插入位置
3. 判断插入位置是否为空，是则直接插入，否则走下一步
4. 判断已有元素和插入元素的哈希值和键值是否相等，相等则覆盖，不相等则走下一步
5. 判断已有元素是链表节点还是树节点，然后遍历链表或树，查找有无 key 相同的节点，有则覆盖，没有则新增
6. 判断是否要树化和反树化
7. 判断是否要扩容

### Get一个元素的流程

1. 根据键的哈希值和HashMap容量计算数组下标
2. 查看数组小标下第一个节点是否为空，空则返回null，否则下一步
3. 查看第一个节点的键是否相同，相同则返回该节点的元素，不同则下一步
4. 第二个节点是链表节点还是树节点，分别执行各自查找键相同的节点的方法
5. 查找键相同的节点则返回该节点的元素，没有则返回null

# 1.7

## 内部属性

```java
static final Entry<?,?>[] EMPTY_TABLE = {};

transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

transient int size;

// If table == EMPTY_TABLE then this is the initial capacity at which the
// table will be created when inflated.
int threshold;

final float loadFactor;

transient int modCount;

transient int hashSeed = 0;

static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;
    ...
}
```

## 构造方法

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);

    this.loadFactor = loadFactor;
    threshold = initialCapacity;
    // 空方法
    init();
}
```

## put(K key, V value)

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        // 初始化数组
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        // 遍历链表，找到key相等节点，直接替换value，然后返回
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            // 留给 LinkedHashMap 实现的方法
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}

/**
 * 初始化数组
 */
private void inflateTable(int toSize) {
    // Find a power of 2 >= toSize
    int capacity = roundUpToPowerOf2(toSize);

    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];

    // 根据情况初始化 hashSeed 的值
    initHashSeedAsNeeded(capacity);
}
final boolean initHashSeedAsNeeded(int capacity) {
    boolean currentAltHashing = hashSeed != 0;
    // 取JVM参数，如果缺失则Holder.ALTERNATIVE_HASHING_THRESHOLD为Integer.MAX_VALUE
    boolean useAltHashing = sun.misc.VM.isBooted() &&
        (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    boolean switching = currentAltHashing ^ useAltHashing;
    if (switching) {
        hashSeed = useAltHashing
            ? sun.misc.Hashing.randomHashSeed(this)
            : 0;
    }
    return switching;
}

private V putForNullKey(V value) {
    // key为null的元素统一放到table[0]节点
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        // 搜索 table[0] 的链表中有无key为null的节点，有则直接替换
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        // 扩容
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    // 新增节点
    createEntry(hash, key, value, bucketIndex);
}
void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    // 头插法
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

## roundUpToPowerOf2(int number)

```java
/**
 * 向上取整到2的n次方
 */
private static int roundUpToPowerOf2(int number) {
    // assert number >= 0 : "number must be non-negative";
    return number >= MAXIMUM_CAPACITY
        ? MAXIMUM_CAPACITY
        : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
```

## hash(Object k)

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

## resize(**int** newCapacity)

```java
/**
 * 扩容
 */
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    // 重新哈希
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                // 重新计算元素哈希值
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            // 头插法
            e.next = newTable[i];
            newTable[i] = e;
            // 链表的下一个元素
            e = next;
        }
    }
}
```

## indexFor(int h, int length)

```java
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
```

## remove(Object key)

```java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}
final Entry<K,V> removeEntryForKey(Object key) {
    if (size == 0) {
        return null;
    }
    int hash = (key == null) ? 0 : hash(key);
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    // 遍历链表，找到key相等的节点进行删除
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e)
                table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }

    return e;
}
```

## get(Object key)

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}
private V getForNullKey() {
    if (size == 0) {
        return null;
    }
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    // 遍历链表，找到key相等的节点返回
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

