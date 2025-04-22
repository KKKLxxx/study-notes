# JDK8、JDK11新特性

## 一、JDK8 

### 1、lambda表达式

当一个接口中只有一个方法时，称为单方法接口，可以用lambda表达式重写方法以简化代码

举例：`(x, y) -> x < y`

相当于捕获两个参数`x`、`y`，返回`x < y`的结果

### 2、Stream

任意Java对象的序列，与List的不同之处在于Stream输出的元素可能并不存在与内存中，而是实时计算出来的（惰性计算：需要获取结果的时候再计算）

应用：`ArrayList<Integer> res`转`int[]`

```java
res.stream().mapToInt(Integer::valueOf).toArray();
```

还有一种比较特殊的`parallelStream`，它是基于`ForkJoin`框架实现的

```java
public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    numbers.parallelStream().forEach(num -> System.out.println(Thread.currentThread().getName() + ">>" + num));
}
```

### 3、default关键字

在接口中，可以用`default`实现默认方法。防止在接口中添加新方法时，要重新修改之前实现过该接口的所有类

```java
public interface Interface1 {
    default void def() {
        System.out.println("default方法");
    }
}
```

### 4、哈希表结构的变化

由之前的 **数组+链表** 转为 **数组+链表+红黑树**

### 5、JVM方法区实现方法的改变

由**永久代**改为**元空间**

### 6、快速构造List

```java
List<String> nameList = Stream.of("Bob", "Alice", "Tim").collect(Collectors.toList());
List<String> nameList2 = Arrays.asList("Bob", "Alice", "Tim");
```

## 二、JDK11

### 1、var关键字

自动推断**局部变量**的类型

```java
int a = 1;
var b = a + 10;
```

 这样就可以自动推断出b的类型为int

### 2、String的方法增强

`isBlank()`：可以判断字符串是否为空或全为空格

`strip()`：相比`trim()`，`strip()`可以删除中文空格

### 3、快速构造List/Map

```java
List<String> nameList3 = List.of("Bob", "Alice", "Tim");
Map<Integer, Integer> map = Map.of(1, 2, 3, 4);
```

