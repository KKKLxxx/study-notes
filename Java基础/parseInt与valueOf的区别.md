# parseInt与valueOf的区别

查看源码

```
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}
```

````
public static Integer valueOf(String s) throws NumberFormatException {
    return Integer.valueOf(parseInt(s, 10));
}
````

可以看到，valueOf实际上是调用了parseInt，不过返回值的类型有一些区别，这就涉及到了自动拆箱与自动装箱

众所周知，自动拆箱与自动装箱会对性能有一些影响。如果我们最终需要的类型就是int，那么最好直接使用parseInt，否则可以选择valueOf