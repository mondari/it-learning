[TOC]

## add 方法

分析 ArrayList 源码先从 add 方法开始

```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

上述步骤主要有两步：

1. ensureCapacityInternal 方法保证内部数组容量足够插入新元素，不够会自动扩容，扩容后容量为原来容量的1.5倍。
2. 把新元素插入到数组后面

ensureCapacityInternal 方法内部使用 grow 方法进行扩容：

```java
/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 保证新容量为旧容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    // 将旧数组复制到容量更大的副本
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

