# JDK新特性

## 一、JDK8 

### 1. lambda表达式

当一个接口中只有一个方法时，称为单方法接口，可以用lambda表达式重写方法以简化代码

举例：`(x, y) -> x < y`

相当于捕获两个参数`x`、`y`，返回`x < y`的结果

常用于简写Runnable接口，若用匿名内部类实现：

```java
public class AnonymousClassExample {
	public static void main(String[] args) {
		Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Running using anonymous class");
            }
        });
        t1.start();
    }
}
```

用lambda简写：

```java
public class LambdaExample {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> System.out.println("Running using lambda expression"));
        t1.start();
    }
}
```

### 2. Stream

Stream是一种适用于集合对象的数据处理方式，可以提高代码的可读性和简洁性，还能利用多核处理器的优势进行并行处理

常用API：

先定义一个List

```java
List<String> nameList = Stream.of("Bob", "Alice", "Tim").collect(Collectors.toList());
```

#### 2.1 构造Stream

```java
Stream<String> stream1 = Arrays.stream(new String[] {"A", "B", "C"}); // 数组，需要Arrays的静态方法
Stream<String> stream2 = nameList.stream(); // 集合，直接调用stream()方法
```

因为Java的范型不支持基本类型，所以我们无法用`Stream<int>`这样的类型，会发生编译错误。为了保存`int`，只能使用`Stream<Integer>`，但这样会产生频繁的装箱、拆箱操作。为了提高效率，Java标准库提供了`IntStream`、`LongStream`和`DoubleStream`这三种使用基本类型的`Stream`，它们的使用方法和范型`Stream`没有大的区别，设计这三个`Stream`的目的是提高运行效率：

```java
// 将int[]数组变为IntStream:
IntStream is = Arrays.stream(new int[] {1, 2, 3});
// 将Stream<String>转换为LongStream:
LongStream ls = List.of("1", "2", "3").stream().mapToLong(Long::parseLong);
```

#### 2.2 map()

map()用于将一个stream转换为另一个stream

```java
Stream<Integer> s = Stream.of(1, 2, 3, 4, 5).map(n -> n * n);
```

这里lambda表达式`n -> n * n`可以将1~5分别平方，新的stream中存有1, 4, 9, 16, 25

也可以传入一个函数引用

```java
nameList.stream().map(String::trim).map(String::toLowerCase);
```

则对nameList中的每个元素调用了trim()和toLowerCase()方法

#### 2.3 filter()

filter()用于过滤stream中不符合条件的元素

```java
nameList.stream().filter(name -> name.startsWith("A"))
```

这样stream中会只剩下以'A'开头的元素

#### 2.4 limit()

limit()可以截取前几个元素

```java
nameList.stream().limit(2)
```

#### 2.5 skip()

skip()可以跳过几个元素

```java
nameList.stream().skip(1).limit(2)
```

可以配合limit()使用

#### 2.6 reduce()

reduce()是一个聚合方法

```java
int res = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(0, (sum, n) -> sum + n)
```

这样就将1~9的和算出来了。其中`reduce(0, (sum, n) -> sum + n)`中的`0`可以看做sum的初始值；`n`可以看做stream中的每个元素；`sum + n`就是每次迭代后的返回值，会赋值给sum

如果计算乘积的话也是同理

```java
int res = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(1, (product, n) -> product * n);
```

但是要注意初始值需要更改为1

#### 2.7 collect()

collect()可以将stream转换为集合

1、转为List

```java
List<String> list = stream.collect(Collectors.toList());
```

2、转为数组

```java
String[] array = stream.toArray(String[]::new);
```

- List转为数组

  ```java
  String[] array = list.toArray(new String[0]);
  ```

- 数组转为List

  ```java
  List<String> list = Arrays.asList(array);
  ```

3、转为Set

```java
Set<String> set = stream.collect(Collectors.toSet());
```

4、转为Map

```java
Stream<String> stream = Stream.of("APPL:Apple", "MSFT:Microsoft");
Map<String, String> map = stream.collect(Collectors.toMap(
                            // 把元素s映射为key:
                            s -> s.substring(0, s.indexOf(':')),
                            // 把元素s映射为value:
                            s -> s.substring(s.indexOf(':') + 1)));
                        
System.out.println(map); // {MSFT=Microsoft, APPL=Apple}
```

需要分别指定key / value

5、分组输出

```java
List<String> list = Arrays.asList("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
Map<String, List<String>> groups = list.stream()
                .collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));
                
System.out.println(groups); // {A=[Apple, Avocado, Apricots], B=[Banana, Blackberry], C=[Coconut, Cherry]}
```

`groupingBy`中第一个参数指定分组依据，这里将首字母相同的字符串分为一组

#### 2.9 parallelStream()

并行流就是将源数据分为多个子流对象进行多线程操作，然后将处理的结果再汇总为一个流对象，底层基于Fork/Join

```java
public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    numbers.parallelStream().forEach(num -> System.out.println(Thread.currentThread().getName() + ">>" + num));
}
```

### 3. default关键字

在接口中，可以用`default`实现默认方法。防止在接口中添加新方法时，要重新修改之前实现过该接口的所有类

```java
public interface Interface1 {
    default void def() {
        System.out.println("default方法");
    }
}
```

### 4. 哈希表结构的变化

由之前的 **数组+链表** 转为 **数组+链表+红黑树**

### 5. JVM方法区实现方法的改变

由**永久代**改为**元空间**

## 二、JDK9

### 1. 快速构造List/Map

```java
List<String> nameList3 = List.of("Bob", "Alice", "Tim");
Map<Integer, Integer> map = Map.of(1, 2, 3, 4);		// k1=1,v1=2; k2=3,v2=4...
```

根据`.of`方法构造的集合是**不可变集合**，无法对集合进行修改操作

## 三、JDK11

### 1. var关键字

自动推断**局部变量**的类型

```java
int a = 1;
var b = a + 10;
```

 这样就可以自动推断出b的类型为int

### 2. String的方法增强

```java
// 判断字符串是否为空
" ".isBlank();				// true
// 去除字符串首尾空格
" Java ".strip();			// "Java"，相比trim()，strip()可以删除中文空格
// 去除字符串首部空格
" Java ".stripLeading();    // "Java "
// 去除字符串尾部空格
" Java ".stripTrailing();   // "Java"
// 重复字符串多少次
"Java".repeat(3);           // "JavaJavaJava"
```

## 四、JDK17

### 1. 增强的伪随机数生成器

JDK 17 之前，我们可以借助 `Random`、`ThreadLocalRandom`和`SplittableRandom`来生成随机数。不过，这 3 个类都各有缺陷，且缺少常见的伪随机算法支持

Java 17 为伪随机数生成器 （pseudorandom number generator，PRNG，又称为确定性随机位生成器）增加了新的接口类型和实现，使得开发者更容易在应用程序中互换使用各种 PRNG 算法

> [PRNG](https://ctf-wiki.org/crypto/streamcipher/prng/intro/) 用来生成接近于绝对随机数序列的数字序列。一般来说，PRNG 会依赖于一个初始值，也称为种子，来生成对应的伪随机数序列。只要种子确定了，PRNG 所生成的随机数就是完全确定的，因此其生成的随机数序列并不是真正随机的。

使用示例：

```java
RandomGeneratorFactory<RandomGenerator> l128X256MixRandom = RandomGeneratorFactory.of("L128X256MixRandom");
// 使用时间戳作为随机数种子
RandomGenerator randomGenerator = l128X256MixRandom.create(System.currentTimeMillis());
// 生成随机数
randomGenerator.nextInt(10);
```

## 五、JDK21

### 1. 虚拟线程

虚拟线程（Virtual Thread）是 JDK 而不是 OS 实现的轻量级线程（Lightweight Process，LWP），由 JVM 调度。许多虚拟线程共享同一个操作系统线程，虚拟线程的数量可以远大于操作系统线程的数量

在引入虚拟线程之前，`java.lang.Thread` 包已经支持所谓的平台线程，也就是没有虚拟线程之前，我们一直使用的线程。JVM 调度程序通过平台线程来管理虚拟线程，一个平台线程可以在不同的时间执行不同的虚拟线程（多个虚拟线程挂载在一个平台线程上），当虚拟线程被阻塞或等待时，平台线程可以切换到执行另一个虚拟线程

虚拟线程、平台线程和系统内核线程的关系图如下所示：

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202602031411294.png" alt="虚拟线程、平台线程和系统内核线程的关系" style="zoom:67%;" />

相比较于平台线程来说，虚拟线程是廉价且轻量级的，使用完后立即被销毁，因此它们不需要被重用或池化，每个任务可以有自己专属的虚拟线程来运行。虚拟线程暂停和恢复来实现线程之间的切换，避免了上下文切换的额外耗费，兼顾了多线程的优点，简化了高并发程序的复杂，可以有效减少编写、维护和观察高吞吐量并发应用程序的工作量

虚拟线程在其他多线程语言中已经被证实是十分有用的，比如 Go 中的 Goroutine、Erlang 中的进程

