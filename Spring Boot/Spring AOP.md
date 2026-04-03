# Spring AOP

## 一、Spring AOP

### 1. 什么是 AOP

AOP（Aspect Oriented Programming，面向切面编程）是一种编程思想，用于**将与业务逻辑无关的代码抽离出来，从而达到解耦、复用的目的。如权限检验、日志收集等**

### 2. AOP 的核心概念

#### 2.1 JoinPoint（连接点）

连接点是可以被增强的代码，在理论上的 AOP（比如 AspectJ）中，连接点可以是：

- 方法调用
- 构造器调用
- 字段访问
- 异常抛出

但在 Spring AOP 中，连接点 = 方法调用（仅限 public 方法）

#### 2.2 Pointcut（切点）

切点定义了连接点的匹配规则，通常定义为被注解修饰的方法

也可以定义为限制在某个包下的类：

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

这个表达式的意思是，匹配 service 包下的所有方法，这些被匹配到的连接点，就变成了“目标连接点”

#### 2.3 Advice（通知）

通知就是在原始业务逻辑上增强的逻辑，如权限校验、日志收集等

通知类型包括：

| 类型     | 注解            | 执行时机   |
| -------- | --------------- | ---------- |
| 前置通知 | @Before         | 方法执行前 |
| 后置通知 | @After          | 方法执行后 |
| 环绕通知 | @Around         | 包裹方法   |
| 返回通知 | @AfterReturning | 正常返回后 |
| 异常通知 | @AfterThrowing  | 抛异常后   |

#### 2.4 Aspect（切面）

切面是切点 + 通知的组合

#### 2.5 Weaving（织入）

织入就是插入切面，使切面生效的过程

### 3. AOP 的实现原理

Spring AOP 基于：

- **JDK 动态代理**：有接口
- **CGLIB 动态代理**：无接口

本质都是生成代理对象，在方法前后插入增强逻辑

### 4. AOP 的实现方式

**要通过注解来配置AOP**

#### 4.1 添加依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
</dependency>
```

#### 4.2 定义注解

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

##### 4.2.1 @Target

表示注解的**作用域**，ElemenetType参数包括

* ElemenetType.CONSTRUCTOR：构造器声明
* ElemenetType.FIELD：域声明（包括 enum 实例） 
* ElemenetType.LOCAL_VARIABLE：局部变量声明
* **ElemenetType.METHOD：方法声明**
* ElemenetType.PACKAGE：包声明
* ElemenetType.PARAMETER：参数声明
* **ElemenetType.TYPE：类，接口（包括注解类型）或enum声明**

比如如果为`METHOD`，说明该注解应该用在一个方法上；如果是`TYPE`，说明该注解应该用在一个类/接口上

##### 4.2.2 @Retention

表示注解的**生命周期**。可选的RetentionPolicy参数包括

* RetentionPolicy.SOURCE：注解将被编译器丢弃
* RetentionPolicy.CLASS：注解在class文件中可用，但会被JVM丢弃
* **RetentionPolicy.RUNTIME：JVM将在运行期也保留注解，因此可以通过反射机制读取注解的信息**

如果我们自己编写注解，最常用的是`RUNTIME`

#### 4.3 定义切面

**切面是切点Pointcut + 通知Advice的组合**

**切点定义了连接点的匹配规则，通常定义为被注解修饰的方法**

**通知就是在原始业务逻辑上增强的逻辑，如权限校验、日志收集等**

多个切面的执行顺序可通过`@Order`控制

```java
@Order(3)	// 值越小优先级越高
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

#### 4.4 使用注解

```java
@OperationLogger(value = "重置用户密码")
public String restPwd() {

}
```

通过在方法上添加`@OperationLogger(value = "重置用户密码")`，就可以实现执行`restPwd()`方法的同时进行日志收集

## 二、AspectJ

### 1. AspectJ 是什么

AspectJ 是一个完整的 AOP 框架，它通过直接修改字节码的方式在编译阶段增强方法（而非动态代理），从而不依赖Spring容器，支持更多的连接点，并提供更快的运行速度

Spring AOP 借用了 AspectJ 的语法，但底层使用的是动态代理，Spring 使用 `@Aspect` ≠ 使用 AspectJ

### 2. 对比

| 对比项                                  | Spring AOP             | AspectJ    |
| --------------------------------------- | ---------------------- | ---------- |
| 实现原理                                | 动态代理               | 字节码织入 |
| 织入时机                                | 运行期                 | 编译期     |
| 使用限制                                | 只能在Spring框架中使用 | 无框架限制 |
| 是否支持private/static/final/构造器方法 | 否                     | 是         |
| 运行速度                                | 慢                     | 快         |

但大部分情况下Spring AOP已经足够，除非不适用Spring框架，或者要增强特殊方法

