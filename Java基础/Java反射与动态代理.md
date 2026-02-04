# Java反射与动态代理

## 一、反射

反射之中包含了一个「反」字，所以想要解释反射就必须先从「正」开始解释

一般情况下，我们使用某个类时必定知道它是什么类，是用来做什么的。于是我们直接对这个类进行实例化，之后使用这个类对象进行操作

```java
Apple apple = new Apple(); // 直接初始化，「正射」
apple.setPrice(4);
```

上面这样子进行类对象的初始化，我们可以理解为「正」

而反射则是一开始并不知道我要初始化的类对象是什么，自然也无法使用`new`关键字来创建对象了

这时候，我们使用 JDK 提供的反射 API 进行反射调用：

```java
Class clz = Class.forName("com.chenshuyi.reflect.Apple");
Method method = clz.getMethod("setPrice", int.class);
Constructor constructor = clz.getConstructor();
Object object = constructor.newInstance();
method.invoke(object, 4);
```

上面两段代码的执行结果，其实是完全一样的。但是其思路完全不一样，第一段代码在未运行时就已经确定了要运行的类（Apple），而第二段代码则是在运行时通过字符串值才得知要运行的类（com.chenshuyi.reflect.Apple）

所以反射就是一种**在程序运行时，动态地获取类的信息并操作类或对象的能力**

**实现方法**：JVM在第一次加载某个类时会生成一个Class对象，里面记录了这个类的信息

**使用场景**：

- **动态代理**：通过反射在运行时创建代理对象

- **Spring注解**：通过反射在运行时检查类、方法、字段上的注解信息，并执行相应逻辑（如@Value加载配置文件）

- **ORM（对象关系映射）框架**：通过反射来映射数据库表和Java对象之间的关系

**反射的缺陷**：

* **性能**：因为编译器不知道程序的真正作用，所以编译器（JIT）无法对代码进行优化
* **安全限制**：反射要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，那么就无法运用反射
* **内部暴露**：反射破坏了抽象性，它允许访问类的私有变量或方法

## 二、动态代理

**动态代理的作用**：将与业务逻辑无关的代码抽离出来，从而达到解耦、复用的目的。如日志收集、权限检验等

**反射在动态代理中的应用**：由于知道被代理类的字段、方法等信息，才可以通过代理类执行被代理类的方法

动态代理的实现有两种：JDK代理和CGLIB代理

### 1. JDK代理

**实现方法**：JDK代理是基于反射实现的。它使代理类继承于一个Proxy类，从而持有InvocationHandler属性。对代理类对象调用的方法都会交由InvocationHandler的invoke方法处理。invoke方法接受3个参数：代理对象、方法、参数列表。重写invoke方法便可以在原方法的基础上添加其他处理逻辑

一个JDK代理的简单实现：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 定义一个接口
interface MyInterface {
    void fun();
}

// 用一个类去实现这个接口
class Person implements MyInterface {
    @Override
    public void fun() {
        System.out.println("Person实现接口方法");
    }
}

// 用一个类实现InvocationHandler接口
class MyInvocationHandler<T> implements InvocationHandler {
    // InvocationHandler持有的被代理对象
    T target;

    public MyInvocationHandler(T target) {
        this.target = target;
    }

    /**
     * proxy:代表动态代理对象
     * method：代表正在执行的方法
     * args：代表调用目标方法时传入的实参
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("代理执行" + method.getName() + "方法"); // 在原方法的基础上添加其他逻辑
        Object result = method.invoke(target, args); // 通过invoke方法调用原方法
        return result;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person();
        InvocationHandler invocationHandler = new MyInvocationHandler<>(person);
        MyInterface proxy = (MyInterface) Proxy.newProxyInstance(Person.class.getClassLoader(),
                Person.class.getInterfaces(), invocationHandler);
        proxy.fun();
    }
}
```

输出结果：

```
代理执行fun方法
Person实现接口方法
```

**缺陷**：只能代理接口方法，因为JDK代理需要继承一个Proxy类，又由于Java的单继承机制，导致代理类无法继承父类的函数，只能实现接口

### 2. CGLIB代理

**实现方式**：CGLIB代理是基于字节码实现的。它使代理类直接继承于被代理类，并通过字节码生成代理类，所以可以实现被代理类的类方法而非仅仅接口方法

一个简单的CGLIB代理实现：

```java
public class Main {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Car.class);
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy)
                    throws Throwable {
                System.out.println("before");
                Object res = methodProxy.invokeSuper(obj, args);
                System.out.println("after");
                return res;
            }
        });
        Car car = (Car) enhancer.create();
        car.print();
    }
}

class Car {
    void print() {
        System.out.println("执行原方法");
    }
}
```

由于CGLIB并非JDK自带，所以需要通过Maven引入一个依赖

```java
<dependency>
    <groupId>org.sonatype.sisu.inject</groupId>
    <artifactId>cglib</artifactId>
    <version>3.1.1</version>
</dependency>
```

输出结果：

```
before
执行原方法
after
```

### 3. 总结

| 代理方式     | JDK代理                 | CGLIB代理                                    |
| ------------ | ----------------------- | -------------------------------------------- |
| **使用限制** | 仅可代理接口方法        | 可代理接口方法与类方法                       |
| **性能**     | 一般更快                | 由于涉及字节码操作，所以更慢，尤其在创建类时 |
| **依赖**     | `java.lang.reflect`自带 | 需要引入CGLIB库                              |
| **实现方式** | 反射                    | 字节码                                       |

### 4. Spring AOP对代理方式的选择

- 当被代理类存在接口方法时，选择JDK代理。但如果被代理类还有类方法，则类方法无法被增强（因为JDK代理原生、简单、快）
- 当被代理类不存在接口方法时，选择CGLIB代理

可以通过配置强制使用CGLIB代理，或者将类方法提取到接口中，从而实现类方法的增强

