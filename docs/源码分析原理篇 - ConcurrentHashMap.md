# 1.7

1.7 版本的ConcurrentHashMap 基于数组+链表，使用Segment+锁来降低锁的粒度，不同 Segment 之间无线程同步问题，同一个 Segment 才需要加锁同步，从而提高并发性能。

## 内部属性

```java
/**
 * Mask value for indexing into segments. The upper bits of a
 * key's hash code are used to choose the segment.
 */
final int segmentMask;

/**
 * Shift value for indexing within segments.
 */
final int segmentShift;

/**
 * The segments, each of which is a specialized hash table.
 * 分段，每段都是个哈希表
 * 注意 segments 不会扩容，只有其内部哈希表才会扩容
 */
final Segment<K,V>[] segments;
```

其中 Segment 的定义如下

```java
/**
 * Segment 类继承了ReentrantLock，说明其具有加锁同步功能
 */
static final class Segment<K,V> extends ReentrantLock implements Serializable {

	/**
	 * The per-segment table. Elements are accessed via
	 * entryAt/setEntryAt providing volatile semantics.
	 */
	transient volatile HashEntry<K,V>[] table;

	/**
	 * The number of elements. Accessed only either within locks
	 * or among other volatile reads that maintain visibility.
	 */
	transient int count;

	/**
	 * The total number of mutative operations in this segment.
	 * Even though this may overflows 32 bits, it provides
	 * sufficient accuracy for stability checks in CHM isEmpty()
	 * and size() methods.  Accessed only either within locks or
	 * among other volatile reads that maintain visibility.
	 */
	transient int modCount;

	/**
	 * The table is rehashed when its size exceeds this threshold.
	 * (The value of this field is always <tt>(int)(capacity *
	 * loadFactor)</tt>.)
	 */
	transient int threshold;

	final float loadFactor;
}
```

其中 HashEntry 的定义如下

```java
static final class HashEntry<K,V> {
	final int hash;
	final K key;
	volatile V value;
	volatile HashEntry<K,V> next;
}
```

## Unsafe 机制

```java
// Unsafe mechanics
private static final sun.misc.Unsafe UNSAFE;
private static final long SBASE;
private static final int SSHIFT;
private static final long TBASE;
private static final int TSHIFT;
private static final long HASHSEED_OFFSET;
private static final long SEGSHIFT_OFFSET;
private static final long SEGMASK_OFFSET;
private static final long SEGMENTS_OFFSET;

static {
    int ss, ts;
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class tc = HashEntry[].class;
        Class sc = Segment[].class;
        TBASE = UNSAFE.arrayBaseOffset(tc);
        SBASE = UNSAFE.arrayBaseOffset(sc);
        ts = UNSAFE.arrayIndexScale(tc);
        ss = UNSAFE.arrayIndexScale(sc);
        HASHSEED_OFFSET = UNSAFE.objectFieldOffset(
            ConcurrentHashMap.class.getDeclaredField("hashSeed"));
        SEGSHIFT_OFFSET = UNSAFE.objectFieldOffset(
            ConcurrentHashMap.class.getDeclaredField("segmentShift"));
        SEGMASK_OFFSET = UNSAFE.objectFieldOffset(
            ConcurrentHashMap.class.getDeclaredField("segmentMask"));
        SEGMENTS_OFFSET = UNSAFE.objectFieldOffset(
            ConcurrentHashMap.class.getDeclaredField("segments"));
    } catch (Exception e) {
        throw new Error(e);
    }
    if ((ss & (ss-1)) != 0 || (ts & (ts-1)) != 0)
        throw new Error("data type scale not a power of two");
    SSHIFT = 31 - Integer.numberOfLeadingZeros(ss);
    TSHIFT = 31 - Integer.numberOfLeadingZeros(ts);
}
```

## 构造方法

```java
/**
 * 初始化 segments
 *
 * @param initialCapacity 初始容量，默认16
 * @param loadFactor 负载因子，默认0.75f
 * @param concurrencyLevel 并发级别，默认16，控制segments数组大小
 */
@SuppressWarnings("unchecked")
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    // 变量 sshift 是幂，ssize 是 segment 数量
    // ssize=2^sshift，ssize>=concurrencyLevel
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    this.segmentShift = 32 - sshift;
    this.segmentMask = ssize - 1;
    
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 哈希表大小=容量÷segment数量
    int c = initialCapacity / ssize;
    // 向上取整
    if (c * ssize < initialCapacity)
        ++c;
    // 确保哈希表大小是2的n次方
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;
    
    // create segments and segments[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

整体流程如下：

1. 根据 concurrencyLevel 计算segment数量
2. 根据容量和segment数量计算哈希表大小
3. 初始化segment数组和数组第一个元素



## put(K key, V value)

```java
public V put(K key, V value) {
	Segment<K,V> s;
	if (value == null)
		throw new NullPointerException();
	int hash = hash(key);
	int j = (hash >>> segmentShift) & segmentMask;
	if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
		 (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        // 确保Segment已初始化，因为一开始只初始化过segments[0]
		s = ensureSegment(j);
    // 调用 Segment 的 put 方法
	return s.put(key, hash, value, false);
}

static final class Segment<K,V> extends ReentrantLock implements Serializable {
    
	final V put(K key, int hash, V value, boolean onlyIfAbsent) {
        // put和remove操作都要加锁
		HashEntry<K,V> node = tryLock() ? null :
			scanAndLockForPut(key, hash, value);
		V oldValue;
		try {
			HashEntry<K,V>[] tab = table;
			int index = (tab.length - 1) & hash;
			HashEntry<K,V> first = entryAt(tab, index);
			for (HashEntry<K,V> e = first;;) {
				if (e != null) {
					K k;
					if ((k = e.key) == key ||
						(e.hash == hash && key.equals(k))) {
						oldValue = e.value;
						if (!onlyIfAbsent) {
							e.value = value;
                            // 注意跟HashMap不同，这里替换旧元素也要modCount+1
							++modCount;
						}
						break;
					}
					e = e.next;
				}
				else {
                    // 没找到key相等的元素，则插入新元素
					if (node != null)
						node.setNext(first);
					else
                        // 头插法
						node = new HashEntry<K,V>(hash, key, value, first);
					int c = count + 1;
					if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                        // 扩容+重新哈希，然后再插入新节点
						rehash(node);
					else
                        // 头插法
						setEntryAt(tab, index, node);
					++modCount;
					count = c;
					oldValue = null;
					break;
				}
			}
		} finally {
			unlock();
		}
		return oldValue;
	}
}
```

### scanAndLockForPut

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    
    /**
     * Scans for a node containing given key while trying to
     * acquire lock, creating and returning one if not found. Upon
     * return, guarantees that lock is held. UNlike in most
     * methods, calls to method equals are not screened: Since
     * traversal speed doesn't matter, we might as well help warm
     * up the associated code and accesses as well.
     *
     * @return a new node if key not found, else null
     */
    private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
        HashEntry<K,V> first = entryForHash(this, hash);
        HashEntry<K,V> e = first;
        HashEntry<K,V> node = null;
        int retries = -1; // negative while locating node
        while (!tryLock()) {
            HashEntry<K,V> f; // to recheck first below
            if (retries < 0) {
                if (e == null) {
                    if (node == null) // speculatively create node
                        node = new HashEntry<K,V>(hash, key, value, null);
                    retries = 0;
                }
                else if (key.equals(e.key))
                    retries = 0;
                else
                    e = e.next;
            }
            else if (++retries > MAX_SCAN_RETRIES) {
                lock();
                break;
            }
            else if ((retries & 1) == 0 &&
                     (f = entryForHash(this, hash)) != first) {
                e = first = f; // re-traverse if entry changed
                retries = -1;
            }
        }
        return node;
    }   
}
```

### rehash(HashEntry<K,V> node)

扩容+重新哈希，并插入新节点

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    
	/**
	 * Doubles size of table and repacks entries, also adding the
	 * given node to new table
	 */
	@SuppressWarnings("unchecked")
	private void rehash(HashEntry<K,V> node) {
		/*
		 * Reclassify nodes in each list to new table.  Because we
		 * are using power-of-two expansion, the elements from
		 * each bin must either stay at same index, or move with a
		 * power of two offset. We eliminate unnecessary node
		 * creation by catching cases where old nodes can be
		 * reused because their next fields won't change.
		 * Statistically, at the default threshold, only about
		 * one-sixth of them need cloning when a table
		 * doubles. The nodes they replace will be garbage
		 * collectable as soon as they are no longer referenced by
		 * any reader thread that may be in the midst of
		 * concurrently traversing table. Entry accesses use plain
		 * array indexing because they are followed by volatile
		 * table write.
		 */
		HashEntry<K,V>[] oldTable = table;
		int oldCapacity = oldTable.length;
		int newCapacity = oldCapacity << 1;
		threshold = (int)(newCapacity * loadFactor);
		// 创建新数组
		HashEntry<K,V>[] newTable =
			(HashEntry<K,V>[]) new HashEntry[newCapacity];
		// sizeMask用于配合哈希值求下标
		int sizeMask = newCapacity - 1;
		// 遍历旧数组
		for (int i = 0; i < oldCapacity ; i++) {
			HashEntry<K,V> e = oldTable[i];
			if (e != null) {
				HashEntry<K,V> next = e.next;
				int idx = e.hash & sizeMask;
				// 单节点直接插入到新数组
				if (next == null)   //  Single node on list
					newTable[idx] = e;
				else { // Reuse consecutive sequence at same slot
					HashEntry<K,V> lastRun = e;
					int lastIdx = idx;
					// 遍历链表，直到最后一个重新哈希后下标不一样的节点，这个节点之后的所有节点是要放在一起
					for (HashEntry<K,V> last = next;
						 last != null;
						 last = last.next) {
						int k = last.hash & sizeMask;
						if (k != lastIdx) {
							lastIdx = k;
							lastRun = last;
						}
					}
					// lastRun节点及其之后的节点组成的链表统一放到lastIdx位置，不重新哈希，节约时间
					newTable[lastIdx] = lastRun;
					// Clone remaining nodes
					// lastRun节点之前的节点则重新哈希
					for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
						V v = p.value;
						int h = p.hash;
						int k = h & sizeMask;
						HashEntry<K,V> n = newTable[k];
						// 头插法
						newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
					}
				}
			}
		}
        
        // 哈希表扩容后，需要将入参node这个新节点添加上去
        // 头插法
		int nodeIndex = node.hash & sizeMask; // add the new node
		node.setNext(newTable[nodeIndex]);
		newTable[nodeIndex] = node;
		table = newTable;
	}
}
```

## ensureSegment(int k)

```java
/**
 * 初始化Segment
 */
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        Segment<K,V> proto = ss[0]; // use segment 0 as prototype
        int cap = proto.table.length;
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
            == null) { // recheck
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // CAS+自旋，直到当前线程或其他线程操作成功则退出循环
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```

## segmentForHash 和 entryForHash

```java
/**
 * Get the segment for the given hash
 */
@SuppressWarnings("unchecked")
private Segment<K,V> segmentForHash(int h) {
	long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
	return (Segment<K,V>) UNSAFE.getObjectVolatile(segments, u);
}

/**
 * Gets the table entry for the given segment and hash
 */
@SuppressWarnings("unchecked")
static final <K,V> HashEntry<K,V> entryForHash(Segment<K,V> seg, int h) {
	HashEntry<K,V>[] tab;
	return (seg == null || (tab = seg.table) == null) ? null :
		(HashEntry<K,V>) UNSAFE.getObjectVolatile
		(tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
}
```

## remove(Object key)

```java
public V remove(Object key) {
    int hash = hash(key);
    Segment<K,V> s = segmentForHash(hash);
    return s == null ? null : s.remove(key, hash, null);
}

static final class Segment<K,V> extends ReentrantLock implements Serializable {

    final V remove(Object key, int hash, Object value) {
        // put和remove操作都要加锁
        if (!tryLock())
            scanAndLock(key, hash);
        V oldValue = null;
        try {
            HashEntry<K,V>[] tab = table;
            int index = (tab.length - 1) & hash;
            HashEntry<K,V> e = entryAt(tab, index);
            HashEntry<K,V> pred = null;
            while (e != null) {
                K k;
                HashEntry<K,V> next = e.next;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    V v = e.value;
                    if (value == null || value == v || value.equals(v)) {
                        if (pred == null)
                            setEntryAt(tab, index, next);
                        else
                            pred.setNext(next);
                        ++modCount;
                        --count;
                        oldValue = v;
                    }
                    break;
                }
                pred = e;
                e = next;
            }
        } finally {
            unlock();
        }
        return oldValue;
    }
}
```

### scanAndLock

```java
private void scanAndLock(Object key, int hash) {
    // similar to but simpler than scanAndLockForPut
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    int retries = -1;
    while (!tryLock()) {
        HashEntry<K,V> f;
        if (retries < 0) {
            if (e == null || key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f;
            retries = -1;
        }
    }
}
```

## get(Object key)

```java
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // 先定位Segment
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        // 再定位HashEntry
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
             (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

## size()

```java
public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    // 先不加锁统计两次，如果有改动则再统计，还是有改动则加锁统计
    
    final Segment<K,V>[] segments = this.segments;
    
    // 变量size是统计结果，overflow标记统计结果是否溢出
    // sum累计各Segment的改动次数，last是上一次sum，两者配合可判断是否有改动
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    // 累加sum
                    sum += seg.modCount;
                    // 累加size
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            // 统计到第二遍才有可能相等
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```

# 1.8

1.8 版本的 ConcurrentHashMap 基于数组+链表+红黑树，使用 CAS+synchronized 来提高并发性能。

## 内部属性

ConcurrentHashMap 内部属性不仅比 HashMap 多，且均用 volatile 修饰。

```java
/**
 * 内部数组。
 */
transient volatile Node<K,V>[] table;

/**
 * The next table to use; non-null only while resizing.
 */
private transient volatile Node<K,V>[] nextTable;

/**
 * Base counter value, used mainly when there is no contention,
 * but also as a fallback during table initialization
 * races. Updated via CAS.
 */
private transient volatile long baseCount;

/**
 * 数组初始化和调整大小控制。如果为负数，则表示正在初始化数组或调整大小：-1表示正在初始化，否则-（1+活动的调整大小线程数）。
 * 另外，未初始化时，该值表示初始容量，为0表示使用默认容量16。
 * 初始化后，保存下一个元素计数值，根据该计数值调整数组的大小。
 */
private transient volatile int sizeCtl;

/**
 * The next table index (plus one) to split while resizing.
 */
private transient volatile int transferIndex;

/**
 * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
 */
private transient volatile int cellsBusy;

/**
 * Table of counter cells. When non-null, size is a power of 2.
 */
private transient volatile CounterCell[] counterCells;

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    // 下面两个属性比HashMap多了个volatile修饰
    volatile V val;
    volatile Node<K,V> next;
}
```

## Unsafe 机制

```java
// Unsafe mechanics
private static final sun.misc.Unsafe U;
private static final long SIZECTL;
private static final long TRANSFERINDEX;
private static final long BASECOUNT;
private static final long CELLSBUSY;
private static final long CELLVALUE;
private static final long ABASE;
private static final int ASHIFT;

static {
    try {
        U = sun.misc.Unsafe.getUnsafe();
        Class<?> k = ConcurrentHashMap.class;
        SIZECTL = U.objectFieldOffset
            (k.getDeclaredField("sizeCtl"));
        TRANSFERINDEX = U.objectFieldOffset
            (k.getDeclaredField("transferIndex"));
        BASECOUNT = U.objectFieldOffset
            (k.getDeclaredField("baseCount"));
        CELLSBUSY = U.objectFieldOffset
            (k.getDeclaredField("cellsBusy"));
        Class<?> ck = CounterCell.class;
        CELLVALUE = U.objectFieldOffset
            (ck.getDeclaredField("value"));
        
        Class<?> ak = Node[].class;
        ABASE = U.arrayBaseOffset(ak);
        int scale = U.arrayIndexScale(ak);
        if ((scale & (scale - 1)) != 0)
            throw new Error("data type scale not a power of two");
        ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
    } catch (Exception e) {
        throw new Error(e);
    }
}

static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

## 构造方法

```java
public ConcurrentHashMap() {
}

public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    // 数组未初始化前，sizeCtl 表示容量
    this.sizeCtl = cap;
}
```

## put(K key, V value)

```java
public V put(K key, V value) {
	return putVal(key, value, false);
}
/**
 * @param onlyIfAbsent 表示只有下标位置不存在key相等的元素才插入。
 */
final V putVal(K key, V value, boolean onlyIfAbsent) {
	if (key == null || value == null) throw new NullPointerException();
	int hash = spread(key.hashCode());
    // binCount 表示链表长度
	int binCount = 0;
	for (Node<K,V>[] tab = table;;) {
        // 变量f是头节点，n是数组长度，i是下标，fh是头节点哈希值
		Node<K,V> f; int n, i, fh;
        
		if (tab == null || (n = tab.length) == 0)
            // 插入第一个元素时，先初始化数组
			tab = initTable();
        
        //头结点为空，直接CAS插入
		else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // CAS成功则跳出循环，失败则继续循环
            // 下次循环不会再走这里，因为已有节点了
			if (casTabAt(tab, i, null,
						 new Node<K,V>(hash, key, value, null)))
				break;                   // no lock when adding to empty bin
		}
		else if ((fh = f.hash) == MOVED)
			tab = helpTransfer(tab, f);
		else {
			V oldVal = null;
            // 以头节点作为锁，锁粒度更低
			synchronized (f) {
				if (tabAt(tab, i) == f) {
                    // 哈希值>=0为链表节点，小于0有其它含义，见下面
					if (fh >= 0) {
						binCount = 1;
                        // 遍历链表，直到key相等则替换，或下一个节点为空则插入
						for (Node<K,V> e = f;; ++binCount) {
							K ek;
							if (e.hash == hash &&
								((ek = e.key) == key ||
								 (ek != null && key.equals(ek)))) {
								oldVal = e.val;
								if (!onlyIfAbsent)
									e.val = value;
								break;
							}
							Node<K,V> pred = e;
							if ((e = e.next) == null) {
								pred.next = new Node<K,V>(hash, key,
														  value, null);
								break;
							}
						}
					}
                    // 哈希值为-2时，表示红黑树根节点
					else if (f instanceof TreeBin) {
						Node<K,V> p;
                        // 设置为2的意义何在?
						binCount = 2;
						if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
													   value)) != null) {
							oldVal = p.val;
							if (!onlyIfAbsent)
								p.val = value;
						}
					}
				}
			}
            //
			if (binCount != 0) {
				if (binCount >= TREEIFY_THRESHOLD)
					treeifyBin(tab, i);
				if (oldVal != null)
					return oldVal;
				break;
			}
		}
	}
    // 元素个数+1？
	addCount(1L, binCount);
	return null;
}

/*
 * Encodings for Node hash fields. See above for explanation.
 */
static final int MOVED     = -1; // hash for forwarding nodes
static final int TREEBIN   = -2; // hash for roots of trees
static final int RESERVED  = -3; // hash for transient reservations
```

从上面可以看成，put 操作是先通过数组下标找到头节点，为空直接CAS插入，否则给头结点加锁，然后遍历链表，找到key相等的节点进行替换，或找到下一个节点为空的节点进行插入。

## spread(int h)

```java
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

/**
 * 类似于 HashMap 的 hash(Object key) 方法，多了个“& HASH_BITS”操作
 */
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```

由该方法可以看出，ConcurrentHashMap不支持存储key为空的元素。

## initTable()

```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
	Node<K,V>[] tab; int sc;
	while ((tab = table) == null || tab.length == 0) {
        // sizeCtl < 0，说明其它线程在初始化，让出线程调度
		if ((sc = sizeCtl) < 0)
			Thread.yield(); // lost initialization race; just spin
		else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
			try {
				if ((tab = table) == null || tab.length == 0) {
					int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
					@SuppressWarnings("unchecked")
					Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
					table = tab = nt;
                    // 其实就是 0.75 * n，表示阈值吗?
					sc = n - (n >>> 2);
				}
			} finally {
				sizeCtl = sc;
			}
			break;
		}
	}
	return tab;
}
```

## helpTransfer(Node<K,V>[] tab, Node<K,V> f)

```java
/**
 * Helps transfer if a resize is in progress.
 */
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
	Node<K,V>[] nextTab; int sc;
	if (tab != null && (f instanceof ForwardingNode) &&
		(nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        // 记下此时的 resizeStamp
		int rs = resizeStamp(tab.length);
		while (nextTab == nextTable && table == tab &&
			   (sc = sizeCtl) < 0) {
			if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
				sc == rs + MAX_RESIZERS || transferIndex <= 0)
				break;
			if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                // 重新哈希
				transfer(tab, nextTab);
				break;
			}
		}
		return nextTab;
	}
	return table;
}

static final class ForwardingNode<K,V> extends Node<K,V> {
	final Node<K,V>[] nextTable;
	ForwardingNode(Node<K,V>[] tab) {
		super(MOVED, null, null, null);
		this.nextTable = tab;
	}
}

private static int RESIZE_STAMP_BITS = 16;

static final int resizeStamp(int n) {
    return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
}
```

## *transfer

重新哈希

## *addCount

加减元素个数？

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    // counterCells 初始值为空
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```

## *fullAddCount

## remove(Object key)

```java
public V remove(Object key) {
    return replaceNode(key, null, null);
}
final V replaceNode(Object key, V value, Object cv) {
    int hash = spread(key.hashCode());
    for (Node<K,V>[] tab = table;;) {
        // 变量f是头结点，n是数组长度，i是下标，fh是头结点哈希值
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0 ||
            (f = tabAt(tab, i = (n - 1) & hash)) == null)
            break;
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            boolean validated = false;
            // 1.给头节点加锁
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        validated = true;
                        // 2.遍历链表
                        // 变量 pred 为上一个节点
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;
                            // 3.找到key相等的节点
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                V ev = e.val;
                                // 删除操作中cv是空的
                                if (cv == null || cv == ev ||
                                    (ev != null && cv.equals(ev))) {
                                    oldVal = ev;
                                    // 删除操作中value是空的
                                    if (value != null)
                                        e.val = value;
                                    else if (pred != null)
                                        // 4.非头结点，则替换上一个节点的next
                                        pred.next = e.next;
                                    else
                                        // 4.头结点，则直接替换
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            pred = e;
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    else if (f instanceof TreeBin) {
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;
                        if ((r = t.root) != null &&
                            (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            if (cv == null || cv == pv ||
                                (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    p.val = value;
                                else if (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            if (validated) {
                if (oldVal != null) {
                    // value == null 说明是删除操作
                    if (value == null)
                        // 元素个数-1？
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    return null;
}
```

## get(Object key)

```java
public V get(Object key) {
	Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
	int h = spread(key.hashCode());
	if ((tab = table) != null && (n = tab.length) > 0 &&
		(e = tabAt(tab, (n - 1) & h)) != null) {
        // 同一个下标，哈希值不是肯定相等的吗？
		if ((eh = e.hash) == h) {
			if ((ek = e.key) == key || (ek != null && key.equals(ek)))
				return e.val;
		}
        // hash < 0 说明在扩容或是红黑树节点
		else if (eh < 0)
            // 为啥要分开找？
			return (p = e.find(h, key)) != null ? p.val : null;
		while ((e = e.next) != null) {
			if (e.hash == h &&
				((ek = e.key) == key || (ek != null && key.equals(ek))))
				return e.val;
		}
	}
	return null;
}

static class Node<K,V> implements Map.Entry<K,V> {
	/**
	 * Virtualized support for map.get(); overridden in subclasses.
	 * 不同类型的Node，该方法实现不一样
	 */
	Node<K,V> find(int h, Object k) {
		Node<K,V> e = this;
		if (k != null) {
			do {
				K ek;
				if (e.hash == h &&
					((ek = e.key) == k || (ek != null && k.equals(ek))))
					return e;
			} while ((e = e.next) != null);
		}
		return null;
	}
}
```

