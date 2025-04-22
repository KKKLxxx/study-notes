# Java迭代器

## 一、遍历

在Collections集合类下，有三大接口，分别是List, Set, Queue。它们都有一个共同的遍历方式，就是迭代器

```java
List<Integer> list = new ArrayList<>();
for (Iterator<Integer> it = list.iterator(); it.hasNext();) {
    int a = it.next();
}
```

常见的还有foreach遍历方法

```java
for (int a : list) {
    int b = a;
}
```

但foreach底层实际上也是通过迭代器遍历的

迭代器的优点就在于**不用知道集合的底层结构就可以遍历**

而且对于LinkedList类型，传统的get()方法遍历效率是O(n)，而迭代器遍历的效率是O(1)

对于ArrayList，就有三种遍历方式，并且遍历效率排序为：`get() > foreach > 迭代器`

## 二、ConcurrentModificationException（Fail-fast）

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
