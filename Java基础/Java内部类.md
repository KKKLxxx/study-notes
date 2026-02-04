# Java内部类

内部类不能单独存在，必须依赖于它的外部类

## 一、成员内部类

```Java
public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer("Nested"); // 实例化一个Outer
        Outer.Inner inner = outer.new Inner(); // 实例化一个Inner
        inner.hello();
    }
}

class Outer {
    private String name;

    Outer(String name) {
        this.name = name;
    }

    class Inner {
        void hello() {
            System.out.println("Hello, " + Outer.this.name);
        }
    }
}
```

成员内部类让一个内部类作为外部类的成员变量

## 二、局部内部类

```Java
class Outer {
    public void method() {
        class Inner {
        }
    }
}
```

局部内部类让一个内部类定义在外部类的一个方法里

## 三、匿名内部类

```Java
public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer("Nested");
        outer.asyncHello();
    }
}

class Outer {
    private String name;

    Outer(String name) {
        this.name = name;
    }

    void asyncHello() {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello, " + Outer.this.name);
            }
        };
        new Thread(r).start();
    }
}
```

观察`asyncHello()`方法，我们在方法内部实例化了一个`Runnable`。`Runnable`本身是接口，接口是不能实例化的，所以这里实际上是定义了一个实现了`Runnable`接口的匿名类，并且通过`new`实例化该匿名类，然后转型为`Runnable`

## 四、静态内部类

```Java
public class Main {
    public static void main(String[] args) {
        Outer.StaticNested sn = new Outer.StaticNested();
        sn.hello();
    }
}

class Outer {
    private static String NAME = "OUTER";

    private String name;

    Outer(String name) {
        this.name = name;
    }

    static class StaticNested {
        void hello() {
            System.out.println("Hello, " + Outer.NAME);
        }
    }
}
```

根据类加载机制和对象加载机制，可以得到以下2条结论

1、静态内部类的创建不需要依赖于外部对象

2、**静态内部类不能引用外部类的非static变量和方法（因为外部类的非static变量需要依赖于一个外部类对象，静态内部类的加载时间先于外部类对象的加载。如果静态内部类有一个外部类非static变量的引用，则矛盾）**

## 五、作用

### 1. 解决继承及实现接口出现同名方法的问题

编写一个接口 Demo

```Java
public interface Demo {
	void test();
}
```

编写一个类 MyDemo

```Java
public class MyDemo {
    public void test() {
		System.out.println("父类的test方法");
	}
}
```

编写一个[测试类](https://www.zhihu.com/search?q=测试类&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A708467570})

```Java
public class DemoTest extends MyDemo implements Demo {
    public void test() {
    }
}
```

这样的话我就有点懵了，这样如何区分这个方法是接口的还是继承的，所以我们使用内部类解决这个问题

```Java
public class DemoTest extends MyDemo {
    private class inner implements Demo {
        public void test() {
            System.out.println("接口的test方法");
        }
    }
  
    public Demo getIn() {
        return new inner();
    }
  
    public static void main(String[] args) {	// 调用接口而来的test()方法
        DemoTest dt = new DemoTest();
        Demo d = dt.getIn();
        d.test();	// 调用继承而来的test()方法
        dt.test();
    }
}

// 运行结果
接口的test方法
父类的test方法
```

### 2. 实现多继承

```Java
public class Demo1 {
    public String name() {
        return "BWH_Steven";
    }
}
    
public class Demo2 {
    public String email() {
        return "xxx.@163.com";
    }
}
    
public class MyDemo {
    private class test1 extends Demo1 {
        public String name() {
            return super.name();
        }
    }
    
    private class test2 extends Demo2 {
        public String email() {
            return super.email();
        }
    }
    
    public String name() {
        return new test1().name();
    }
    
    public String email() {
        return new test2().email();
    }
    
    public static void main(String args[]) {
        MyDemo md = new MyDemo();
        System.out.println("我的姓名:" + md.name());
        System.out.println("我的邮箱:" + md.email());
    }
}
```