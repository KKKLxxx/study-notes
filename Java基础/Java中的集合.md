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
        Vector(C)
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

### 1. List

#### 1.1 ArrayList

##### 1.1.1 扩容机制

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

##### 1.1.2 ArrayList与数组的转换

- **List转数组**

  toArray()+传入长度为零的数组（若不传入则返回Object[]）

  ```java
  List<String> strList = new ArrayList<>();
  String[] strArr = strList.toArray(new String[0]);
  ```

- **数组转List**

  - **对象数组转List**

    ```java
    String[] strArr = {"a", "b", "c"};
    // 返回固定大小的List（属于Arrays内部类，不可add/remove）
    List<String> strList1 = Arrays.asList(strArr);
    
    // 若需要可变List，包装一层ArrayList
    List<String> strList2 = new ArrayList<>(Arrays.asList(strArr));
    strList2.add("d"); // 正常执行
    ```

  - **基本类型数组转List**

    ```java
    // 错误示例：int[]转List会变成List<int[]>，而非List<Integer>
    int[] numArr = {1, 2, 3};
    List<int[]> wrongList = Arrays.asList(numArr);
    
    // 正确方式1：手动装箱（JDK8-）
    List<Integer> numList1 = new ArrayList<>();
    for (int num : numArr) {
        numList1.add(num);
    }
    
    // 正确方式2：Stream流（JDK8+）
    List<Integer> numList2 = Arrays.stream(numArr).boxed().collect(Collectors.toList());
    ```

#### 1.2 CopyOnWriteArrayList

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

### 2. Set

#### 2.1 HashSet

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

### 3. Queue

#### 3.1 PriorityQueue

`PriorityQueue`的底层结构是堆

## 三、Map

**HashMap的遍历**

```java
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println("Key: " + entry.getKey() + ", Value: " + entry.getValue());
}
```

### 1. LinkedHashMap

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

### 2. TreeMap

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

## 四、线程安全的集合

- **Vector**：线程安全的动态数组，其内部方法基本都经过synchronized修饰，连续存储，相当于线程安全版的ArrayList
- **CopyOnWriteArrayList**：其中所有写操作（add，set等）都通过对底层数组进行全新复制来实现（写操作会加锁）
- **BlockingQueue**：用于生产者-消费者场景
- **Hashtable**：线程安全的哈希表，HashTable 的加锁方法是给每个方法加上 synchronized 关键字，这样锁住的是整个 Table 对象，不支持 null 键和值
- **ConcurrentHashMap**：它与 HashTable 的主要区别是二者加锁粒度的不同

## 五、集合的遍历方式

### 1. 普通 for 循环

```java
for (int i = 0; i < list.size(); i++) {
    String element = list.get(i);
    System.out.println(element);
}
```

### 2. 增强 for 循环（for-each循环）

foreach底层实际上也是通过迭代器遍历的

```java
for (String element : list) {
    System.out.println(element);
}
```

### 3. forEach 方法

```java
list.forEach(element -> System.out.println(element));
```

### 4. 迭代器

迭代器的优点就在于**不用知道集合的底层结构就可以遍历，且可在遍历的同时删除元素**

而且对于LinkedList类型，传统的get()方法遍历效率是O(n)，而迭代器遍历的效率是O(1)

```java
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()) {
    String element = iterator.next();
    System.out.println(element);
}
```

#### 4.1 ConcurrentModificationException（Fail-fast）

`Fail-fast`是快速失败机制，是Java集合中的一种错误检测机制。在通过迭代器遍历**非线程安全的容器**（如ArrayList）时，如果同时调用了**集合自身的**`add()/remove()`方法，就有可能触发`Fail-fast`机制，即抛出`ConcurrentModificationException`异常

`Iterator`接口定义了`next()`方法

```java
public interface Iterator<E> {
	E next();
}
```

不同集合类对`next()`方法有不同的实现，对于非线程安全的容器（如ArrayList），每次调用`next()`前，都会通过`checkForComodification()`方法进行一次版本检测

```java
// ArrayList对next()的实现
public E next() {
    checkForComodification();
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}

final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

`modCount`记录了容器的修改次数。`expectedModCount`值记录的是调用`iterator`方法时的`modCount`值。如果遍历过程中，调用了一次写操作，`modCount`就会自增1，也就与`expectedModCount`不相等了，所以就抛出异常

因为`CopyOnWriteArrayList`是线程安全的，所以不需要通过`checkForComodification()`方法检测，也就不会抛出`ConcurrentModificationException`异常

```java
// CopyOnWriteArrayList对next()的实现
public E next() {
    if (! hasNext())
        throw new NoSuchElementException();
    return (E) snapshot[cursor++];
}
```

安全起见，**如果一定要在迭代器遍历的同时删除元素，那么应当使用迭代器提供的`remove()`方法**

### 5. Stream

```java
list.stream().forEach(element -> System.out.println(element));
```

