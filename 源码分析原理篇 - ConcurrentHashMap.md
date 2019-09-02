[TOC]

## JDK 7 版本的 ConcurrentHashMap

该版本的 ConcurrentHashMap 使用”分段锁“来提高并发性能。”分段锁“就是将一个大的 HashTable 分成小的 Segment，根据 hash(key.hashCode()) 来决定元素存放在哪个 Segment，不同的 Segment 之间无需考虑线程同步问题，同一 Segment 中才需要加锁保证线程同步。

## JDK 8 版本的 ConcurrentHashMap

该版本的 ConcurrentHashMap 不再使用”分段锁“，而是使用 CAS(CompareAndSwap) 乐观锁来提高并发性能。

## 树化条件

注意 ConcurrentHashMap 的树化条件是 `binCount >= TREEIFY_THRESHOLD` ，而 HashMap 的树化条件是 `binCount >= TREEIFY_THRESHOLD - 1` ，TREEIFY_THRESHOLD 是常量8。

## spread 方法解读



