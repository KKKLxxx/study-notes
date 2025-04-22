# ConcurrentHashMap底层原理

一、HashMap与HashTable的缺陷
----------------------

### （一）HashMap

HashMap是非线程安全集合，在多线程环境下，如果没有特殊处理，会由于指令重排序等原因，发生赋值错误甚至死循环

### （二）HashTable

HashTable虽然保证了线程安全，但是由于它每次操作都要锁住整个集合，导致效率极其低下

综合以上两点，ConcurrentHashMap通过**分段加锁**技术，在保证线程安全的同时提高了访问效率

二、ConcurrentHashMap的底层结构
------------------------

[HashMap底层结构、扩容机制](https://gitee.com/KKKLxxx/study-notes/blob/master/Java%E5%9F%BA%E7%A1%80/HashMap%E5%BA%95%E5%B1%82%E7%BB%93%E6%9E%84%E3%80%81%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6.md "HashMap底层结构、扩容机制")

其实ConcurrentHashMap的底层结构与HashMap类似，都是数组+链表+红黑树，所以就不再赘述了

## 三、ConcurrentHashMap如何实现线程安全

### 1、put操作

在添加结点时，首先判断数组是否为空，如果为空，则初始化（通过**CAS操作**保证线程安全）；

如果数组不为空，则判断对应下标的元素是否为空，如果为空，则通过**CAS操作**设置该结点；

如果元素不为空**通过synchronized对链表/红黑树的头结点加锁**，替换或新增该结点，最后判断是否需要转为红黑树

**总结**：当数组为空或者数组当前元素为空时，通过CAS保证；当有其他元素时，冲突概率大，用synchronized效率更高

### 2、get操作

Node结点的结构：

```
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
        ...
}
```

可以看到，val与next都用volatile修饰，这保证了线程间的可见性。由于get只是一个读操作，所以volatile完全可以保证每次读取的数据是最新的

## 四、ConcurrentHashMap的键值为什么不能为null

在put操作的源码中，有判断键值是否为null的代码

```
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
}
```

任何一个为空就抛异常，而在HashMap中是允许为null的

造成这种区别的原因是，ConcurrentHashMap用于多线程场景，一个线程get后如果得到null，**无法判断是真的存在一个null值还是根本不存在这个值**，即有歧义
