# 简介

ArrayList 的底层基于数组，其内部维护着元素的数量。其内部属性如下：

```java
transient Object[] elementData;

private int size;
```



其父类 AbstractList 也有一个重要的属性 modCount，表示“结构化修改”的次数（**“结构化修改”指扩容、删除**），负责实现 Interator 迭代器的 fail-fast 功能。Interator 每次操作时都会通过 checkForComodification 方法检查 modCount 是否变动，一旦变动就会抛出 ConcurrentModificationException。

# 时间复杂度

- get、set 方法的时间复杂度是常量时间

- add(int index, E element)、remove(Object o) 方法的时间复杂度是O(n)

# 构造方法

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
private static final Object[] EMPTY_ELEMENTDATA = {};
// 上面两个的区别
// new ArrayList() 会使用 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 数组，首次添加元素时会扩容为10
// new ArrayList(0) 会使用 EMPTY_ELEMENTDATA 数组，首次添加元素时会扩容为1，扩容效率低

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

# add

add 方法有两个

```java
/**
 * 将元素插入到列表指定位置。需要先将该位置元素和后续元素向右移动
 */
public void add(int index, E element) {
    // 数组越界检查
    rangeCheckForAdd(index);

    // 容量不够会扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    
    // 将指定位置元素和后续元素向右移动
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 插入元素
    elementData[index] = element;
    size++;
}

/**
 * 将指定元素追加到列表后面
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

## ensureCapacityInternal

```java
private void ensureCapacityInternal(int minCapacity) {
    // 计算所需最小容量，不够则扩容
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

/**
 * 计算容量，一般都是返回 minCapacity
 */
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 通过ArrayList()创建的ArrayList，首次添加元素会走这一步，返回DEFAULT_CAPACITY默认容量10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    // modCount 负责实现 Interator 迭代器的 fail-fast 功能
    modCount++;

    // 是否需要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    // 新容量为旧容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    
    // 将旧数组复制到容量更大的数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

# set(int index, E element)

set 与 add 方法的不同之处是只将元素添加到指定位置，并不会引起扩容和将指定位置元素和后续元素向右移动。

```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

# get(int index)

```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```

# remove

remove 方法有两个

```java
public E remove(int index) {
    // 数组越界检查
    rangeCheck(index);

    // modCount 负责实现 Interator 迭代器的 fail-fast 功能
    modCount++;
    E oldValue = elementData(index);

    // 将指定位置后续元素向左移动
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 删除元素左移后最右边多出的元素
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

/**
 * 删除列表中指定元素，只删除出现的第一个
 */
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

/**
 * 相比remove方法，跳过数组越界检查，也不返回删除的元素
 */
private void fastRemove(int index) {
    // modCount 负责实现 Interator 迭代器的 fail-fast 功能
    modCount++;
    
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

# rangeCheckForAdd 和 rangeCheck

两个方法都是数组越界检查，但是检查范围和使用场景不一样。

```java
/**
 * A version of rangeCheck used by add and addAll.
 */
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * 数组越界检查，但不检查 Index < 0 的情况，该情况由数组抛出 ArrayIndexOutOfBoundsException。
 * 主要给 add 和 addAll 外的其它方法使用
 */
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

