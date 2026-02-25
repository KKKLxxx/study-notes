# Spring AOP

## 一、Spring AOP

### 1. 什么是 AOP

AOP（Aspect Oriented Programming，面向切面编程）是一种编程思想，用于**把“通用功能”从业务代码中分离出来，从而简化代码**

例如：

- 日志记录
- 权限校验
- 事务管理
- 性能监控

### 2. 为什么需要 AOP

如果没有 AOP：

```java
public void save() {
    log();
    checkPermission();
    doBusiness();
    commit();
}
```

每个方法都写重复代码

有了 AOP：

```java
public void save() {
    doBusiness();
}
```

日志、权限、事务由 Spring 自动织入

### 3. AOP 的核心概念

#### 3.1 JoinPoint（连接点）

在理论上的 AOP（比如 AspectJ）里，连接点可以是：

- 方法执行
- 方法调用
- 构造器执行
- 字段访问
- 异常抛出

但在 Spring AOP 中，连接点 = 方法执行（仅限 public 方法）

#### 3.2 Pointcut（切点）

切点是从众多连接点中筛选出来的一部分，它定义了一个匹配规则，比如

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

这个表达式的意思是，匹配 service 包下的所有方法，这些被匹配到的连接点，就变成了“目标连接点”

#### 3.3 Advice（通知）

通知是在目标方法执行的某个时机要做的事情

比如：

- 记录日志
- 开启事务
- 校验权限

通知类型包括：

| 类型     | 注解            | 执行时机   |
| -------- | --------------- | ---------- |
| 前置通知 | @Before         | 方法执行前 |
| 后置通知 | @After          | 方法执行后 |
| 返回通知 | @AfterReturning | 正常返回后 |
| 异常通知 | @AfterThrowing  | 抛异常后   |
| 环绕通知 | @Around         | 包裹方法   |

#### 3.4 Aspect（切面）

切面是切点 + 通知的组合，是一个完整的增强模块

#### 3.5 Weaving（织入）

织入是把切面应用到目标对象的过程

### 4. Spring AOP 的实现原理

Spring AOP 基于：

- **JDK 动态代理**：有接口
- **CGLIB 动态代理**：无接口

本质都是生成代理对象，在方法前后插入增强逻辑

## 二、注解

### 1. 注解的作用

注解是“配置 AOP 的方式”

### 2. 注解的使用

#### 2.1 添加依赖

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
</dependency>
```

#### 2.2 定义注解

要通过`@interface`来定义一个注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OperationLogger {
    /**
     * 业务名称
     */
    String value() default "";
}
```

同时有几个元注解

##### 2.2.1 @Target

表示注解的**作用域**，ElemenetType参数包括

* ElemenetType.CONSTRUCTOR：构造器声明
* **ElemenetType.FIELD：域声明（包括 enum 实例）** 
* ElemenetType.LOCAL_VARIABLE：局部变量声明
* **ElemenetType.METHOD：方法声明**
* ElemenetType.PACKAGE：包声明
* ElemenetType.PARAMETER：参数声明
* **ElemenetType.TYPE：类，接口（包括注解类型）或enum声明**

比如如果为`METHOD`，说明该注解应该用在一个方法上；如果是`TYPE`，说明该注解应该用在一个类/接口上

##### 2.2.2 @Retention

表示注解的**生命周期**。可选的RetentionPolicy参数包括

* RetentionPolicy.SOURCE：注解将被编译器丢弃
* RetentionPolicy.CLASS：注解在class文件中可用，但会被VM丢弃
* **RetentionPolicy.RUNTIME：JVM将在运行期也保留注释，因此可以通过反射机制读取注解的信息**

如果我们自己编写注解，最常用的是`RUNTIME`

##### 2.2.3 @Documented

将此注解包含在Javadoc中

##### 2.2.4 @Inherited

允许子类继承父类中的注解

#### 2.3 定义注解逻辑

通过一个切面来定义使用了该注解的方法要执行哪些业务逻辑无关的代码

部分代码：

```java
@Aspect
@Component
public class LoggerAspect {

    @Pointcut(value = "@annotation(operationLogger)")
    public void pointcut(OperationLogger operationLogger) {
        
    }

    @Around(value = "pointcut(operationLogger)")
    public Object doAround(ProceedingJoinPoint joinPoint, OperationLogger operationLogger) throws Throwable {
        // 先执行业务
        Object result = joinPoint.proceed();

        try {
            // 日志收集
            handle(joinPoint);
        } catch (Exception e) {
            log.error("日志记录出错!", e);
        }

        return result;
    }
}
```

#### 2.4 使用注解

```java
@OperationLogger(value = "重置用户密码")
public String restPwd() {
    
}
```

通过在方法上添加`@OperationLogger(value = "重置用户密码")`，就可以实现执行`restPwd()`方法的同时进行日志收集

### 3. 注解何时生效

注解只有被解析之后才会生效，常见的解析方法有两种：

- **编译期直接扫描**：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用`@Override` 注解，编译器在编译的时候就会检测当前的方法是否重写了父类对应的方法
- **运行期通过反射处理**：像框架中自带的注解(比如 Spring 框架的`@Value` 、`@Component`)都是通过反射来进行处理的

在SpringBoot中，通过一个`@SpringBootApplication`注解包含了3个子注解

1、@SpringBootConfiguration：将Application启动类作为Bean注入到Spring容器中

2、@EnableAutoConfiguration：启动自动配置功能

3、@ComponentScan：扫描并加载组件（Component, Controller, Service, Repository等）

执行完这些流程之后就注解就生效了

### 4. 注解失效

#### 4.1 失效场景

在Spring中使用@Transactional、@Cacheable或自定义AOP注解时，会发现：在对象内部的方法中调用该对象的其他使用AOP注解的方法，被调用方法的AOP注解失效。比如

```java
public class UserService{
    @Transactional
    public void hello(){
        try {
            saveUser();
        } catch (Exception e) {
        }
    }
 
    @Transactional
    public void saveUser(){
        User user = new User();
        user.setName("zhangsan");
        // 存入数据库
    }
}
```

在同一个类中的方法，再调用AOP注解（@Transactional注解也是AOP注解）的方法，会使AOP注解失效

此时如果saveUser()存数据库动作失败抛出异常，“存入数据库“动作不会回滚，数据仍旧存入数据库

#### 4.2 失效原因

因为不管是JDK代理还是CGLIB代理，都是通过生成一个代理类，然后在代理类中调用被代理类的方法，在调用原方法之前和之后通过一些操作来完成代理功能（比如保证事务性）。如果嵌套地调用方法，会导致方法被直接调用，而不经过代理类

#### 4.3 解决方法

可以通过`@Autowired`自己注入自己，然后在需要嵌套调用的地方通过注入的bean调用

```java
public class UserService{
	@Autowired
	private UserService userService;
	
    @Transactional
    public void hello(){
        try {
            userService.saveUser();
        } catch (Exception e) {
        }
    }
 
    @Transactional
    public void saveUser(){
        User user = new User();
        user.setName("zhangsan");
        // 存入数据库
    }
}
```

不过最好避免嵌套调用

## 三、AspectJ

### 1. AspectJ 是什么

AspectJ 是一个完整的 AOP 框架，它通过直接修改字节码的方式在编译阶段增强方法（而非动态代理），从而不依赖Spring容器，并提供更快的运行速度

Spring AOP 借用了 AspectJ 的语法，但底层使用的是动态代理，Spring 使用 `@Aspect` ≠ 使用 AspectJ

### 2. 对比

| 对比项                                  | Spring AOP             | AspectJ           |
| --------------------------------------- | ---------------------- | ----------------- |
| 实现方式                                | 动态代理               | 字节码织入        |
| 织入时机                                | 运行期                 | 编译期 / 类加载期 |
| 使用限制                                | 只能在Spring框架中使用 | 无框架限制        |
| 是否支持private/final/static/构造器方法 | 否                     | 是                |
| 运行速度                                | 慢                     | 快                |

但大部分情况下Spring AOP已经足够，除非不适用Spring框架，或者要增强特殊方法

