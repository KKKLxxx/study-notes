# Stream使用方法

先定义一个List

```
List<String> nameList = Stream.of("Bob", "Alice", "Tim").collect(Collectors.toList());
```

## 一、构造Stream

```
Stream<String> stream1 = Arrays.stream(new String[] {"A", "B", "C"}); // 数组，需要Arrays的静态方法
Stream<String> stream2 = nameList.stream(); // 集合，直接调用stream()方法
```

因为Java的范型不支持基本类型，所以我们无法用`Stream<int>`这样的类型，会发生编译错误。为了保存`int`，只能使用`Stream<Integer>`，但这样会产生频繁的装箱、拆箱操作。为了提高效率，Java标准库提供了`IntStream`、`LongStream`和`DoubleStream`这三种使用基本类型的`Stream`，它们的使用方法和范型`Stream`没有大的区别，设计这三个`Stream`的目的是提高运行效率：

```
// 将int[]数组变为IntStream:
IntStream is = Arrays.stream(new int[] {1, 2, 3});
// 将Stream<String>转换为LongStream:
LongStream ls = List.of("1", "2", "3").stream().mapToLong(Long::parseLong);
```

## 二、map()

map()用于将一个stream转换为另一个stream

```
Stream<Integer> s = Stream.of(1, 2, 3, 4, 5).map(n -> n * n);
```

这里lambda表达式`n -> n * n`可以将1~5分别平方，新的stream中存有1, 4, 9, 16, 25

也可以传入一个函数引用

```
nameList.stream().map(String::trim).map(String::toLowerCase);
```

则对nameList中的每个元素调用了trim()和toLowerCase()方法

## 三、filter()

filter()用于过滤stream中不符合条件的元素

```
nameList.stream().filter(name -> name.startsWith("A"))
```

这样stream中会只剩下以'A'开头的元素

## 四、limit()

limit()可以截取前几个元素

```
nameList.stream().limit(2)
```

## 五、skip()

skip()可以跳过几个元素

```
nameList.stream().skip(1).limit(2)
```

可以配合limit()使用

## 六、reduce()

reduce()是一个聚合方法

```
int res = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(0, (sum, n) -> sum + n)
```

这样就将1~9的和算出来了。其中`reduce(0, (sum, n) -> sum + n)`中的`0`可以看做sum的初始值；`n`可以看做stream中的每个元素；`sum + n`就是每次迭代后的返回值，会赋值给sum

如果计算乘积的话也是同理

```
int res = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(1, (product, n) -> product * n);
```

但是要注意初始值需要更改为1

## 七、collect()

collect()可以将stream转换为集合

1、转为List

```
List<String> list = stream.collect(Collectors.toList());
```

2、转为数组

```
String[] array = stream.toArray(String[]::new);
```

- List转为数组

  ```
  String[] array = list.toArray(new String[0]);
  ```

- 数组转为List

  ```
  List<String> list = Arrays.asList(array);
  ```

3、转为Set

```
Set<String> set = stream.collect(Collectors.toSet());
```

4、转为Map

```
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

```
List<String> list = Arrays.asList("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
Map<String, List<String>> groups = list.stream()
                .collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));
                
System.out.println(groups); // {A=[Apple, Avocado, Apricots], B=[Banana, Blackberry], C=[Coconut, Cherry]}
```

`groupingBy`中第一个参数指定分组依据，这里将首字母相同的字符串分为一组

