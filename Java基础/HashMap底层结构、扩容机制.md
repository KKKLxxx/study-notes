# HashMap底层结构、扩容机制

## 一、底层结构

在JDK8之后，HashMap由之前的“数组+链表”转为“**数组+链表+红黑树**”

类似于这种结构

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20210914200053114.png)

数组即左边一列，链表即横着相连的那部分

那红黑树在哪呢？当链表的长度大于等于8时，为了将查找时间复杂度由n降为logn，这个链表就会转化为红黑树

但上面都是很基础的内容，下面讲两个比较不常见的知识点

### 1、链表长度大于等于8时，一定会转为红黑树吗？

在HashMap源码中有这样几个变量

```java
/*
The bin count threshold for using a tree rather than list for a bin. 
Bins are converted to trees when adding an element to a bin with at least this many nodes. 
*/
static final int TREEIFY_THRESHOLD = 8;

/*
The smallest table capacity for which bins may be treeified.
(Otherwise the table is resized if too many nodes in a bin.)
*/
static final int MIN_TREEIFY_CAPACITY = 64;


final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // 省略代码
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
    ...
}
```

`TREEIFY_THRESHOLD = 8`说明确实定义了一个“树化的临界值“，且这个值为8

但在`treeifyBin()`这个函数中，并不是直接把链表转化为红黑树，而是**先检查数组容量是否小于`MIN_TREEIFY_CAPACITY`这个值，如果小于，会先进行扩容**。这样做的目的是**尽量避免将链表转为树这种较为复杂的结构，所以通过扩容的方式减少哈希冲突**，尝试将这个结点分配到数组的其他位置上

如果数组容量大于等于64了，那么就只能将其转为红黑树了

### 2、当红黑树结点数小于8时，会不会转回链表？

```java
/*
The bin count threshold for untreeifying a (split) bin during a
resize operation.
*/
static final int UNTREEIFY_THRESHOLD = 6;
```

直接说结论：会，但不是立刻转回，而是要等到下一次resize()时检验结点数与`UNTREEIFY_THRESHOLD`的关系，如果小于等于6，就可以转回

## 二、扩容机制

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

其实很简单，就是数组初始容量`capacity`为(1 << 4) = 16，加载因子`load_factor`为0.75，当已经使用的容量等于`capacity * load_factor`时，就会调用`resize()`函数，将最大容量扩充为原来的2倍

目的：**减少哈希冲突，提高存取效率**

### 为什么加载因子等于0.75？

因为`load_factor`太大会导致哈希冲突增加，存取效率低

太小会导致数组的利用率低，存放的数据会很分散

`load_factor`的默认值为`0.75f`是官方给出的一个比较好的临界值
