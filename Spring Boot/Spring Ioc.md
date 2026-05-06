# Spring Ioc

## 一、什么是 IoC

IoC（Inversion of Control，控制反转）指的是**对象的创建和依赖关系的管理，不再由程序员手动控制，而交给IoC容器管理**

## 二、不使用 IoC 时

设想一个场景：用户注册功能。在这个场景中，业务类 `UserService` 需要调用数据库操作类 `UserRepository`

在这种模式下，`UserService` 必须亲自动手创建它所需要的依赖对象

```Java
// 1. 数据库操作类
public class UserRepository {
    public void save(String user) {
        System.out.println("保存用户到数据库: " + user);
    }
}

// 2. 业务逻辑类
public class UserService {
    // 痛点：UserService 强耦合了 UserRepository 的具体实现
    private UserRepository repository = new UserRepository();

    public void register(String name) {
        System.out.println("执行注册逻辑...");
        repository.save(name);
    }
}
```

**这种方式的问题：**

- **强耦合：** 如果 `UserRepository` 的构造函数改了（比如需要传入数据库连接池），你必须修改所有 `new` 过它的地方
- **难以测试：** 你没法轻易地把 `UserRepository` 替换成一个“假”的测试对象（Mock）
- **职责不精：** `UserService` 本该只负责业务逻辑，现在还得负责管理对象的生命周期

## 三、使用 IoC 后

在 IoC 模式下，你只需声明“我需要它”，剩下的交给 Spring 容器去“注入”

```Java
// 1. 标记为 Spring 管理的组件
@Component
public class UserRepository {
    public void save(String user) {
        System.out.println("保存用户到数据库: " + user);
    }
}

// 2. 标记为 Service，并通过注解“要”依赖
@Service
public class UserService {
    
    // 关键点：不再自己 new，而是等着 Spring 塞进来
    @Autowired
    private UserRepository repository;

    public void register(String name) {
        System.out.println("执行注册逻辑...");
        repository.save(name);
    }
}
```

**发生了什么变化？**

- **控制权转移：** 创建 `UserRepository` 的权力从 `UserService` 转移到了 Spring 容器手中

- **声明式依赖：** 你只需要通过 `@Autowired` 告诉 Spring 你需要什么，它会在运行时自动把实例注入进去

- **极度灵活：** 如果你想把 `UserRepository` 换成 `MySQLRepository`，只要它们实现同一个接口，你甚至不需要改动 `UserService` 的代码

**在不使用IoC时，如果一个Bean A依赖于另一个Bean B，那么就需要在A中通过构造函数创建B，并且需要A管理B的生命周期，这就造成了代码的耦合。使用IoC后，A只需通过注解告诉Ioc容器它需要B，IoC容器就会自动创建并注入B，并管理B的生命周期**

## 四、依赖注入DI

IoC 是一种思想，指对象的创建和依赖管理由容器负责

而DI（Dependency Injection）是 IoC 具体的实现方式，DI 做的事情是：把创建好的对象交给使用者

例如：

```java
public OrderService(UserService userService) {
    this.userService = userService;
}
```

Spring 在创建 `OrderService` 时，把 `UserService` 传进来，这就是“注入”

有三种依赖注入方式：

### 1. 字段注入（不推荐）

```java
@Service
public class OrderService {

    @Autowired
    private UserService userService;
}
```

### 2. 构造器注入（推荐）

```java
@Service
public class OrderService {

    private final UserService userService;

    @Autowired
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

如果只有一个构造器，`@Autowired` 可以省略

**推荐的原因：**

- **避免空指针**：构造器注入通过构造器保证依赖在被使用前就是已经注入了的，从而避免空指针问题；而字段注入可能在注入前就被使用（但也因此无法解决循环依赖问题，因为构造器需要一个完整的依赖才能注入）
- **保证不可变性**：构造器注入通过final修饰字段，从而保证不可变性；而字段注入本质是先创建对象再赋值，所以无法用final修饰（虽然可通过反射实现，但与语义不符，不推荐这样操作，所以Spring也不支持）
- **便于单元测试**：可通过`OrderService service = new OrderService(mockUserService);`来快速测试，完全不依赖 Spring 容器；而字段注入必须借助反射或 Spring 测试框架

### 3. Setter注入（特定场景可用）

```java
@Service
public class OrderService {

    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

适合Setter注入的场景：

- **可选依赖**：比如缓存是增强功能，没有也能运行

  ```java
  @Autowired(required = false)
  public void setCacheService(CacheService cacheService) {
      this.cacheService = cacheService;
  }
  ```

- **解决循环依赖**：构造器注入无法解决循环依赖，但 Setter 注入可以（因为可以先实例化对象，再注入）

## 五、循环依赖问题

### 1. 什么是循环依赖

循环依赖（Circular Dependency）指：两个或多个 Bean 互相依赖，形成闭环

最简单例子：

```java
@Service
public class A {
    @Autowired
    private B b;
}

@Service
public class B {
    @Autowired
    private A a;
}
```

关系图：

```
A → B
↑   ↓
└───┘
```

### 2. 为什么会出问题

如果是构造器注入：

```java
public A(B b) {}
public B(A a) {}
```

创建 A 时需要 B，创建 B 时又需要 A，两个都“等对方先创建”，导致死锁式失败

### 3. 分情况讨论

#### 3.1 构造器注入的循环依赖（无法解决）

```java
public A(B b) {}
public B(A a) {}
```

Spring 会直接抛异常：

```java
BeanCurrentlyInCreationException
```

原因：构造器必须拿到完整对象，无法提前暴露

#### 3.2 Setter / 字段注入原型Bean的循环依赖（无法解决）

#### 3.3 Setter / 字段注入单例Bean的循环依赖（可以解决）

```java
public class A {
    @Autowired
    private B b;
}

public class B {
    @Autowired
    private A a;
}
```

为什么可以？**因为 Spring 使用了三级缓存机制，通过暴露Bean的早期引用来解决循环依赖**

### 4. 三级缓存机制

#### 4.1 底层原理

Spring 内部维护三个缓存（仅单例 Bean）：

```
一级缓存：singletonObjects（存储Bean的完整引用）

二级缓存：earlySingletonObjects（存储Bean的早期引用，也就是没有完成属性注入的Bean）

三级缓存：singletonFactories（存储创建Bean的工厂，可以用来创建Bean的原始对象或代理对象）
```

创建过程：

1、创建A时发现需要B，把创建A的工厂放入**三级缓存**，转而创建B

2、创建B时发现需要A，通过工厂创建A的早期对象并放入**二级缓存**（根据是否需要AOP决定创建出来的是原始对象还是代理对象）

3、向B注入A在**二级缓存**中的早期引用，完成创建并放入**一级缓存**

4、向A注入B在**一级缓存**中的完整引用，完成创建并放入**一级缓存**

#### 4.2 为什么必须是单例？

因为只有单例才会进入缓存，Prototype（原型/多例）不会被缓存，如果每次都新建，就无法解决

#### 4.3 为什么不能只用二级缓存？

先说结论：**如果没有 AOP，二级缓存可以解决循环依赖；但只要涉及 AOP，就必须使用三级缓存，因为 Spring 需要“延迟决定”暴露的是原始对象，还是代理对象**（如果不管是否需要AOP，直接在二级缓存中放入代理后的对象也行，但是会执行不必要的代理操作）

假设只有二级缓存且A需要AOP，那么B中注入的是A的原始对象，而一级缓存中保存的A的代理对象，导致不一致

## 六、Spring IoC 容器

- `BeanFactory`：基础容器，只提供最核心的对象创建能力，**懒加载**

- `ApplicationContext`：增强版容器，增加了 AOP、事件、国际化、环境管理等企业级功能，**预加载**，实际开发使用，常见实现：
- `ClassPathXmlApplicationContext`
  
- `AnnotationConfigApplicationContext`

