# HashMap底层put与get过程

## 一、put()

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

源码的过程可以简单总结为以下三步

1、通过hash()函数计算key的哈希值，并得到其下标

2、将hash、key、value等信息封装成一个Node

3、根据当前数组位置上的结点情况，采用不同的方法放置此结点。比如是链表还是红黑树，是否需要从链表转为红黑树等

## 二、get()

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

1、通过hash()函数计算key的哈希值，并得到其下标

2、根据当前下标位置上的结点情况，采用不同的方法返回value。如果当前位置没有结点，返回null；如果有一个，直接返回；如果有多个，需要判断是链表还是红黑树，然后**通过equals()方法判断传入的key与当前结点的key是否相同**，不相同则比较下一个结点，相同则返回

## 三、如何确定下标

### 1、hash()函数

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

首先会先检验这个key是否为null，如果为null，那么hash固定为0，所以每个HashMap中只能存在一个key为null的结点。接着通过对key调用hashCode()函数，计算出它的哈希值，并将这个哈希值的高16位与低16位做异或运算，得到最终的哈希值hash

**这个异或运算的作用是让hash值的高16位参与下标的计算，使key的下标分布更均匀。因为数组容量一般不会到达`2^16`这么大，所以如果没有异或运算，hash值的高16位就没有作用了**

### 2、通过哈希值确定下标

```
tab[i = (n - 1) & hash]
```

接着会根据这个式子来确定数组下标，tab就是Node数组，n是数组容量

**举个例子**，当

n = 1 << 4 = 16时，其二进制为

0000 0000 0000 0000 0000 0000 0001 0000（因为是整型，所以一共32位）

n - 1 = 15，其二进制为

0000 0000 0000 0000 0000 0000 0000 1111

假设hash = 43532435，其二进制为

‭0000 0010 1001 1000 0100 0000 1001 0011‬

则(n - 1) & hash的结果二进制为

0000 0000 0000 0000 0000 0000 0000 0011

所以下标为3

这就说明了为什么**HashMap的容量一定要是2的n次幂**，因为由这个确定下标的过程可以看出，只有当容量是2的n次幂时，(n - 1)的二进制的低位才会均为1，这样做与运算时保证了低位都有效，不会出现某个位置不会被使用的情况，**最大化了空间利用率**

**为什么选择与运算而不是模运算呢**？如果用(hash % capicity)当然也能得到相同的结果，但是因为位运算的效率更高，所以当然要选择与运算