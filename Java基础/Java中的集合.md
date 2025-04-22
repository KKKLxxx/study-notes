# Java中的集合

## 一、总览

Java中的集合主要分为两个接口，`Collection`与`Map`，整体结构如下

说明：

- (I)：表示接口
- (C)：表示类
- 层级关系：表示实现接口或者继承类

```
Collection(I):
    List(I):
        ArrayList(C)
        LinkedList(C)
        CopyOnWriteArrayList(C)
    Set(I):
        HashSet(C)
        SortedSet(I):
            TreeSet(C)
    Queue(I):
        PriorityQueue(C)
        Deque(I):
            ArrayDeque(C)
        BlockingQueue(I):
            LinkedBlockingQueue(C)

Map(I):
    HashMap(C):
		LinkedHashMap(C)
    HashTable(C)
    ConcurrentHashMap(C)
    SortedMap(I):
        TreeMap(C)
```

下面挑一些不常见的内容进行介绍

## 二、Collection

### 1、List

#### 1、ArrayList

```java
private static final int DEFAULT_CAPACITY = 10;

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA; // EMPTY_ELEMENTDATA = 0
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}
```

`ArrayList`**在通过无参构造器创建时，会创建一个容量为0的对象，当添加第一个元素后，才会将容量扩展为10**

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

在容量满时添加新元素，会调用`grow()`方法进行扩容。**每次扩容大概会到原来的1.5倍**，因为如果`oldCapacity`是奇数，那么`newCapacity = 1.5 * oldCapacity - 1`

#### 2、CopyOnWriteArrayList

`CopyOnWriteArrayList`是一种**线程安全**的`ArrayList`，适用于**读多写少的并发场景**

**读操作**：直接进行读取，不用加锁。底层数组用volatile保证了线程可见性

```java
private transient volatile Object[] array;

public E get(int index) {
    return get(getArray(), index);
}
```

**写操作**：比如向容器中添加一个元素，则首先将当前数组复制一份，然后在新数组上执行写操作，之后再将原数组覆盖

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock; // 通过读写锁保证写操作的安全
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1); // 复制一个新数组
        newElements[len] = e;
        setArray(newElements); // 替换原数组
        return true;
    } finally {
        lock.unlock();
    }
}
```

**优点**：保证了线程安全

**缺点**：

1、内存占用问题：由于写操作要复制原数组，所以有内存占用问题

2、可能读取到旧数据：在数据已经写入新数组，但还没来得及覆盖时进行读操作，则会读取到旧数据

### 2、Set

#### 1、HashSet

`HashSet`的底层是`HashMap`

```java
private transient HashMap<E,Object> map;

// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();

public HashSet() {
    map = new HashMap<>();
}
```

观察`add()`方法

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

`HashMap`是键值对结构，而`HashSet`只存在键，值会用一个全局唯一的`Object`对象代替

为什么不直接设为`null`呢？况且设为`null`还能够减少内存占用

这是因为对于`HashSet`的`add()`方法，调用了`HashMap`的`put()`方法，该方法的返回值有两种结果

- null：如果不存在对应的key
- 原value：如果存在对应的key

所以**如果在`HashSet`中以`null`为值，就无法判断对应的`key`之前是否存在**

对于`HashSet`中的`remove()`方法同理，`HashMap`中`remove()`方法的返回值同`add()`

### 3、Queue

#### 1、PriorityQueue

`PriorityQueue`的底层结构是堆

## 三、Map

### 1、LinkedHashMap

`LinkedHashMap`在`HashMap`的基础上，通过**双向链表**来维护元素的插入顺序，从而解决`HashMap`不能保持遍历顺序和插入顺序一致的问题

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

`LinkedHashMap`最常见的使用场景就是用来做LRU缓存，`accessOrder`字段用来指定是否根据读取操作更新节点的位置，如果用来做LRU缓存，则`accessOrder`字段应当为`true`

### 2、TreeMap

`TreeMap`通过**红黑树**来将节点根据`key`（默认）进行排序

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;
    
    ......
}
```

所以`TreeMap`适合用于需要排序的场景

