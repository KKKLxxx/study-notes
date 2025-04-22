# Spring循环依赖

## 一、产生场景

就是A对象依赖了B对象，B对象依赖了A对象。

比如：

```java
class A {
    public B b;
}

class B {
    public A a;
}
```

那么循环依赖是个问题吗？

如果不考虑Spring，循环依赖并不是问题，因为对象之间相互依赖是很正常的事情。

比如

```Plaintext
A a = new A();
B b = new B();

a.b = b;
b.a = a;
```

这样，A,B就依赖上了。

但是，在Spring中循环依赖就是一个问题了，为什么？

因为，在Spring中，一个对象并不是简单new出来了，而是会经过一系列的Bean的生命周期，就是因为Bean的生命周期所以才会出现循环依赖问题。当然，在Spring中，出现循环依赖的场景很多，有的场景Spring自动帮我们解决了，而有的场景则需要程序员来解决

## 二、解决方案

### 1、三级缓存

**singletonObjects**：缓存某个beanName对应的经过了完整生命周期的bean

**earlySingletonObjects**：缓存提前拿原始对象进行了AOP之后得到的代理对象

**singletonFactories**：缓存的是一个ObjectFactory，主要用来去生成原始对象进行了AOP之后得到的代理对象，在每个Bean的生成过程中，都会提前暴露一个工厂，这个工厂可能用到，也可能用不到，如果没有出现循环依赖依赖本bean，那么这个工厂无用，本bean按照自己的生命周期执行，执行完后直接把本bean放入singletonObjects中即可，如果出现了循环依赖依赖了本bean，则另外那个bean执行ObjectFactory提交得到一个AOP之后的代理对象(如果有AOP的话，如果无需AOP，则直接得到一个原始对象)

### 2、创建过程

创建A -> 发现需要B -> **把A的早期对象放入singletonFactories** -> 创建B -> 发现需要A -> **singletonFactories创建A的早期对象并放入earlySingletonObjects** -> B创建完成并放入**singletonObjects** -> 继续创建A -> 注入B -> A创建完成

### 3、为什么需要三级

从上面这个分析过程中可以得出，只需要一个缓存就能解决循环依赖了，那么为什么Spring中还需要singletonFactories呢？

这是难点，基于上面的场景想一个问题：**如果A的原始对象注入给B的属性之后，A的原始对象进行了AOP产生了一个代理对象，此时就会出现，对于A而言，它的Bean对象其实应该是AOP之后的代理对象，而B的a属性对应的并不是AOP之后的代理对象，这就产生了冲突**

## 三、不能解决的循环依赖

### 1、通过构造函数产生的

```Java
@Component
public class ObjectA {
    private ObjectB b;
    public ObjectA(ObjectB b) {
        this.b = b;
    }
}
    
@Component
public class ObjectB {
    private  ObjectA a;
    public ObjectB(ObjectA a) {
        this.a = a;
    }
}
```

当我们使用构造方法注入时，准备执行A对象的构造方法，但是构造方法属性依赖了B对象，必须先创建完B对象才能完成构造方法的执行，当创建B调用B对象的构造方法时，发现又依赖于A对象，所以B对象的构造方法也无法执行完毕，得先创建A，结果我们会发现对象A和对象B都无法完成构造方法的执行，那么此时两个对象都没法申请到一个内存地址，当对象内存都生成不了试想又如何为其依赖的属性赋值，显然不可能赋值为null。所以这种情况的循环依赖是没办法解决的，因为始终都没办法解决谁先创建的问题

### 2、原型bean

跟上一种情况类似，原型bean每次都会创建新的对象，无法引用到同一个对象

## 四、参考链接

https://zhuanlan.zhihu.com/p/163031798

https://juejin.cn/post/6985337310472568839#heading-0