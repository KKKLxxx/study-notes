# HashMap实现原理

## 一、底层结构

在JDK8之后，HashMap由之前的“数组+链表”转为“**数组+链表+红黑树**”

类似于这种结构

![](https://raw.githubusercontent.com/KKKLxxx/img-host/master/20210914200053114.png)

数组即左边一列，链表即横着相连的那部分

那红黑树在哪呢？当链表的长度大于等于8时，为了将查找时间复杂度由n降为logn，这个链表就会转化为红黑树（不用平衡二叉树AVL是因为，AVL虽然查询效率相对更高，但代价是插入删除时需要做更多的旋转操作以保持平衡，而**红黑树整体效率更高**）

但上面都是很基础的内容，下面讲两个比较不常见的知识点

### 1. 链表长度大于等于8时，一定会转为红黑树吗？

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

### 2. 当红黑树结点数小于8时，会不会转回链表？

```java
/*
The bin count threshold for untreeifying a (split) bin during a
resize operation.
*/
static final int UNTREEIFY_THRESHOLD = 6;
```

直接说结论：会，但不是立刻转回，而是要等到下一次resize()时检验结点数与`UNTREEIFY_THRESHOLD`的关系，如果小于等于6，就可以转回

### 3. 哈希冲突解决方法有哪些？

- **链地址法**：对哈希值冲突的结点用一个链表/红黑树连接（Java中的HashMap就是用这种方法解决的）

- **开放定址法**：一直往后直到找到一个空的位置

- **再哈希法**：再次计算哈希值

- **公共溢出区**：用一个公共地址保存冲突元素

## 二、扩容机制

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

其实很简单，就是数组初始容量`capacity`为(1 << 4) = 16，加载因子`load_factor`为0.75，当已经使用的容量等于`capacity * load_factor`时，就会调用`resize()`函数，将最大容量扩充为原来的2倍

目的：**减少哈希冲突，提高存取效率**

### 1. 为什么加载因子等于0.75？

因为`load_factor`太大会导致哈希冲突增加，存取效率低

太小会导致数组的利用率低，存放的数据会很分散

`load_factor`的默认值为`0.75f`是官方给出的一个比较好的临界值

## 三、put()执行过程

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

## 四、get()执行过程

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

2、根据当前下标位置上的结点情况，采用不同的方法返回value

- 如果当前位置没有结点，返回null
- 如果有一个，直接返回
- 如果有多个，需要判断是链表还是红黑树，然后**通过equals()方法判断传入的key与当前结点的key是否相同**，不相同则比较下一个结点，相同则返回

## 五、如何确定元素下标

### 1. hashCode()与equals()

#### 1.1 hashCode()与equals()的作用

`equals()`容易理解，就是**用来比较两个对象的属性是否相等**

而`hashCode()`的规定就是：在Java应用程序执行期间，在同一对象上多次调用`hashCode()`方法时，必须一致地返回相同的整数，前提是对象上`equals()`比较中所用的信息没有被修改。如果根据`equals()`方法，两个对象是相等的，那么在两个对象中的每个对象上调用`hashCode()`方法都必须生成相同的整数结果

说白了`hashCode()`就是为对象生成一个哈希值，这个哈希值的作用之一就是**用来确定这个对象作为key时在HashMap中的下标**

#### 1.2 何时需要重写

首先要知道为什么需要重写：**因为这两个函数的默认方法都是根据对象的地址做计算和比较**

假设有一个`Person`类，只含有一个`name`属性。一般来讲，如果两个`Person`对象的`name`属性相同，那么对这两个对象调用`equals()`应该返回`true`。但是如果没有重写方法，这两个对象的地址一定不相同，使用`equals()`比较后会返回`false`

所以**如果要将自定义类作为HashMap的key时，就需要重写**，否则会发生内存泄漏

#### 1.3 如何重写

重写最重要的原则就是：

- 尽管`hashCode()`相等，`equals()`也不一定相等
- 如果`equals()`相等，那么`hashCode()`一定相等（否则HashMap可以把属性相同的多个对象当作不同的key，数据无法正确覆盖，发生内存溢出）

```java
public class Main {
    public static void main(String[] args) {
        HashMap<Person, Integer> map = new HashMap<>();
        Person a = new Person("张三");
        Person b = new Person("张三");
        map.put(a, 1);
        map.put(b, 2);
        System.out.println(map.size());
        System.out.println(a.equals(b));
    }
}

class Person {
    public String name;

    public Person(String name) {
        this.name = name;
    }

    @Override
    public int hashCode() {
        return name.length();
    }

    @Override
    public boolean equals(Object other) {
        return name.equals(((Person)other).name);
    }
}
```

输出结果 

```
1
true
```

 说明b可以覆盖a，如果把两个重写的函数注释掉，输出的结果就会是

```
2
false
```

当然这里只是为了方便，简单重写了一下。实际上要考虑的东西还是比较多的，但是就不在这里细说了

而且要注意，**如果要将自定义类作为HashMap的key，hashCode()和equals()都需要重写，不能只重写其中一个**

### 2. hash()函数

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

首先会先检验这个key是否为null，如果为null，那么hash固定为0，所以每个HashMap中只能存在一个key为null的结点。接着通过对key调用hashCode()函数，计算出它的哈希值，并将这个哈希值的高16位与低16位做异或运算，得到最终的哈希值hash

**这个异或运算的作用是让hash值的高16位参与下标的计算，使key的下标分布更均匀。因为数组容量一般不会到达`2^16`这么大，所以如果没有异或运算，hash值的高16位就没有作用了**

### 3. 通过哈希值确定下标

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

### 4. 哪种类型的对象适合作为key？

不可变对象，比如String。因为不可变对象的hashCode()与equals()的结果是不变的，避免了key发生变化后，计算得到的数组下标发生变化的情况

## 六、死循环问题

HashMap死循环主要是指JDK7中，HashMap扩容时采用头插法。如果在多线程情况下，2条线程同时进行扩容操作，会造成两个结点的next指针互相指向对方的情况

比如最简单的一个情况，一个链表只有A、B两个结点，线程1和线程2一开始都指向A结点

```
A
↓
B
```

线程1扩容时，会在新数组中颠倒链表的原顺序，变为

```
B
↓
A
```

此时线程1已经完成了扩容，B是头结点，`B.next = A`。但是线程2并不知道，所以线程2又要从A开始进行一番操作，把`A.next`修改为B，这样就会形成一个死循环

---

在JDK8中，扩容已经由头插法改为尾插法，多线程情况下虽然不会造成死循环，但是会造成操作的重复执行。多线程推荐使用ConcurrentHashMap
