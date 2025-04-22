# hashCode()与equals()何时需要重写

## 一、hashCode()与equals()的作用

`equals()`容易理解，就是**用来比较两个对象的属性是否相等**

而`hashCode()`的规定就是：在Java应用程序执行期间，在同一对象上多次调用`hashCode()`方法时，必须一致地返回相同的整数，前提是对象上`equals()`比较中所用的信息没有被修改。如果根据`equals()`方法，两个对象是相等的，那么在两个对象中的每个对象上调用`hashCode()`方法都必须生成相同的整数结果

说白了`hashCode()`就是为对象生成一个哈希值，这个哈希值的作用之一就是**用来确定这个对象作为key时在HashMap中的下标**

## 二、何时需要重写

首先要知道为什么需要重写：**因为这两个函数的默认方法都是根据对象的地址做计算和比较**

假设有一个`Person`类，只含有一个`name`属性。一般来讲，如果两个`Person`对象的`name`属性相同，那么对这两个对象调用`equals()`应该返回`true`。但是如果没有重写方法，这两个对象的地址一定不相同，使用`equals()`比较后会返回`false`

所以**如果要将自定义类作为HashMap的key时，就需要重写**，否则会发生内存泄漏

## 三、如何重写

重写最重要的原则就是：

- 尽管`hashCode()`相等，`equals()`也不一定相等
- 如果`equals()`相等，那么`hashCode()`一定相等

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