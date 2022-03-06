# ThreadLocal

## 什么是 ThreadLocal

ThreadLocal 就是线程本地存储，它和 HashMap 一样都能存储键值对，但它的 key 是 ThreadLocal 对象本身，所以一个 ThreadLocal 对象，只能存一个 value，多出的会因哈希冲突而被覆盖。

## ThreadLocal 的作用

保证多线程的环境下，各个线程的数据互不干扰。

## ThreadLocal 的使用场景

一般用来保存请求上下文（RequestContextHolder）、Security上下文（SecurityContextHolder）、以及多租户环境下的租户ID等数据，然后就能在多个地方获取。

PS：后面带有 ContextHolder 的类，一般都是使用 ThreadLocal 来保存变量，比如 Spring 框架中的 RequestContextHolder、SecurityContextHolder。

## ThreadLocal 的使用

ThreadLocal 需要在初始化的时候就要设置初始值，初始化后对其设值操作不影响其它线程：

```java
ThreadLocal<String> global = ThreadLocal.withInitial(() -> "main-thread-value");// 在主线程创建 ThreadLocal 对象并设置初始值
global.set("new value") // 在主线程操作不影响其它线程
    
new Thread(() -> {
    System.out.println(global.get());// -> main-thread-value，在子线程中去访问，返回结果为初始值
}, "sub-thread").start();
```

如果我们一开始没有设置初始值，那么其初始值就是 null：

```java
ThreadLocal<Object> global = new ThreadLocal<>();// 在主线程创建 ThreadLocal 对象但是没设置初始值
global.set("new value");// 在主线程操作不影响其它线程

new Thread(() -> {
    System.out.println(global.get());// -> null，在子线程中去访问，返回结果为初始值
}, "sub-thread").start();
```

定义多个 ThreadLocal，其值都会保存在线程的 threadLocals 中：

```java
ThreadLocal<String> value1 = ThreadLocal.withInitial(() -> "value1");
ThreadLocal<String> value2 = ThreadLocal.withInitial(() -> "value2");
new Thread(() -> {
    System.out.println(value1.get());// -> value1
    System.out.println(value2.get());// -> value2
    value1.set("sub-value1");
    value2.set("sub-value2");
    System.out.println(value1.get());// -> sub-value1
    System.out.println(value2.get());// -> sub-value2
}, "sub-thread").start();
```



## ThreadLocal 是如何保证各个线程的数据互不干扰

这个就要从 ThreadLocal 的存储位置说起，ThreadLocal 的值是存在 Thread 线程类的 threadLocals 成员对象中，由 ThreadLocal 类去维护，这就保证各个线程的数据互不干扰。

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```



## ThreadLocal 的 set 方法

**ThreadLocal 的 set 方法：**

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);// 拿到当前线程的 ThreadLocalMap 对象
    if (map != null)
        map.set(this, value);// 将 ThreadLocal 对象作为 key 保存到 ThreadLocalMap 中
    else
        createMap(t, value);// 如果 map 为空则新建一个 map
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals; // 返回线程中的 ThreadLocalMap 对象
}
```



**其中 ThreadLocalMap的 set 方法：**

```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);// 计算下标(其实就是 hashCode ÷ length)

    // 遍历下标所在位置及往后的节点（主要作用是在哈希冲突时找到下标往后的一个空节点）
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();// Reference 中的 get 方法。我们在 set 的时候是把 key 存到 Reference 中，所以这里先取出

        if (k == key) {// 如果哈希表中已存在该 key，覆盖
            e.value = value;
            return;
        }

        if (k == null) {// key 为 null 代表已被 gc 回收，需要清理该 value，否则会内存泄露
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    // 哈希冲突时会走这里
    tab[i] = new Entry(key, value);
    int sz = ++size;
    
    // 扩容走这里
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

/**
 * 下标加1
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);// 防止下标越界
}

/**
 * Reference 中的 get 方法
 */
public T get() {
    return this.referent;
}
```

注意，第一次往 ThreadLocal 中存值是先调 createMap 方法，第二次才会调 set 方法。





**其中 ThreadLocal 中的 createMap 方法：**

```java
/**
 * ThreadLocal 中的 createMap 方法
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);// 这个 this 是 ThreadLocal 对象
}

/**
 * ThreadLocalMap 构造函数
 */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];// 初始容量固定为16
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);// 计算 key 在哈希表中的下标
    table[i] = new Entry(firstKey, firstValue);// 新建哈希表中的节点
    size = 1;// 哈希表中的元素数量设为1。以后往 map 中添加 ThreadLocal 对象，其值加1
    setThreshold(INITIAL_CAPACITY);// 设置阈值
}

/**
 * ThreadLocal$ThreadLocalMap$Entry 哈希表节点
 */
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    /**
     * Entry 构造函数
     */
    Entry(ThreadLocal<?> k, Object v) {
        super(k);// 调用 Reference 的构造方法（WeakReference 继承自 Reference）
        value = v;
    }
}

```



## ThreadLocal 的 get 方法

ThreadLocal 的 get 方法：

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);// 拿到当前线程中的 ThreadLocalMap 对象
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();// 创建 map 并设置初始值
}

private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

/**
 * 该方法其实是 ThreadLocal 的 set 方法的变种， 目的是替代 set 方法防止其被重写
 */
private T setInitialValue() {
    T value = initialValue();// 拿到 ThreadLocal.withInitial() 的初始值，默认是 null
    Thread t = Thread.currentThread();// 这里以下跟 ThreadLocal 的 set 方法大体相同
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

总结：

1. 拿到当前线程中的 ThreadLocalMap 对象
2. 如果为空，说明没有保存过值，

## ThreadLocal 内存泄露

使用 ThreadLocal 需要注意内存泄露的问题。

### 为什么会内存泄露？

我们看 ThreadLocal$ThreadLocalMap$Entry 节点的代码可知，key（也就是 ThreadLocal 对象）被保存到了 WeakReference 对象中，这就导致在 GC 时，没有外部强引用的 ThreadLocal 会被回收，ThreadLocalMap 中就会出现一个 key=null 的 Entry ，而这个 key=null 的 Entry 是无法访问的，如果这个线程一直没有结束的话，value 就一直无法回收，造成内存泄露。

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}

public class WeakReference<T> extends Reference<T> {

    /**
     * 创建一个引用给定对象的虚引用对象
     * 
     * Creates a new weak reference that refers to the given object.  The new
     * reference is not registered with any queue.
     *
     * @param referent object the new weak reference will refer to
     */
    public WeakReference(T referent) {
        super(referent);
    }

}
```

### 如何避免内存泄露

1. 将 ThreadLocal 变量定义成静态变量，这样就一直存在 ThreadLocal 的强引用，ThreadLocal 也就不会被回收

2. 使用完 ThreadLocal 之后，要调用 remove 方法清除数据

```java
ThreadLocal<String> local = new ThreadLocal<>();
try {
    local.set("xxx");
} finally {
    local.remove();
}

```



## 参考

[ThreadLocal的使用及原理分析](https://mp.weixin.qq.com/s/C4a1dp8iCTLK-NR3DM9gOQ)





