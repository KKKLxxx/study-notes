# ConcurrentHashMap底层原理

一、HashTable与ConcurrentHashMap的区别
----------------------

HashTable采用**全表锁**的机制保证线程安全（对`put()`、`get()`加synchronized锁），但是由于它的每次操作都要锁住整个集合，导致效率极其低下

ConcurrentHashMap采用**分段加锁**的机制保证线程安全，也就是对链表或者红黑树的头结点加锁，所以在保证线程安全的同时提高了访问效率

二、ConcurrentHashMap的底层结构
------------------------

[HashMap底层结构、扩容机制](https://gitee.com/KKKLxxx/study-notes/blob/master/Java%E5%9F%BA%E7%A1%80/HashMap%E5%BA%95%E5%B1%82%E7%BB%93%E6%9E%84%E3%80%81%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6.md "HashMap底层结构、扩容机制")

其实ConcurrentHashMap的底层结构与HashMap类似，都是数组+链表+红黑树，所以就不再赘述了

## 三、ConcurrentHashMap如何实现线程安全

### 1. get操作

Node结点的结构：

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
        ...
}

transient volatile Node<K,V>[] table;
```

可以看到，数组还有每个Node节点的val,next都用volatile修饰，这保证了线程间的可见性。由于get只是一个读操作，所以volatile完全可以保证每次读取的数据是最新的

**总结：读操作是通过用volatile修饰数组还有每个Node节点的val,next来保证线程间的可见性，从而实现不加锁的读取**

### 2. put操作

如果数组下标元素为空，则通过CAS操作设置该结点；

如果元素不为空通过synchronized对链表/红黑树的头结点加锁，替换或新增该结点，最后判断是否需要转为红黑树

**总结：写操作是通过CAS和synchronized来保证的线程安全。当数组元素为空时，冲突概率较小，使用CAS避免加锁；当数组元素不为空时，冲突概率较大，直接使用synchronized对链表或者红黑树的头结点加锁**

**为什么不用写时复制：因为写操作可以原子地修改节点，读线程不会看到中间状态**

## 四、ConcurrentHashMap的键值为什么不能为null

在put操作的源码中，有判断键值是否为null的代码

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
}
```

任何一个为空就抛异常，而在HashMap中是允许为null的

造成这种区别的原因是，ConcurrentHashMap用于多线程场景，一个线程get后如果得到null，**无法判断是真的存在一个null值还是根本不存在这个值**，即有歧义
