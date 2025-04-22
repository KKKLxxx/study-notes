# Spring注解底层原理

## 一、注解的作用

注解（Annotation）可以用于直观地指明一些与业务逻辑无关的数据，在执行该方法时可以通过这些数据进行一些处理，比如日志收集。注解本质上是一种**动态代理**，运用了**反射**机制来获取注解的信息

## 二、注解的使用

### （一）添加依赖

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
</dependency>
```

### （二）定义注解

要通过`@interface`来定义一个注解

```
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

#### 1、@Target

表示注解的**应用范围**，ElemenetType参数包括

* ElemenetType.CONSTRUCTOR 构造器声明
* ElemenetType.FIELD 域声明（包括 enum 实例） 
* ElemenetType.LOCAL_VARIABLE 局部变量声明
* **ElemenetType.METHOD 方法声明**
* ElemenetType.PACKAGE 包声明
* ElemenetType.PARAMETER 参数声明
* **ElemenetType.TYPE 类，接口（包括注解类型）或enum声明**

比如如果为`METHOD`，说明该注解应该用在一个方法上；如果是`TYPE`，说明该注解应该用在一个类/接口上

#### 2、@Retention

表示注解的**生命周期**。可选的RetentionPolicy参数包括

* RetentionPolicy.SOURCE 注解将被编译器丢弃
* RetentionPolicy.CLASS 注解在class文件中可用，但会被VM丢弃
* RetentionPolicy.RUNTIME VM将在运行期也保留注释，因此可以通过反射机制读取注解的信息

如果我们自己编写注解，最常用的是`RUNTIME`

#### 3、@Documented

将此注解包含在Javadoc中

#### 4、@Inherited

允许子类继承父类中的注解

### （三）定义注解逻辑

通过一个切面来定义使用了该注解的方法要执行哪些业务逻辑无关的代码

部分代码：

```
@Aspect
@Component
public class LoggerAspect {

    @Pointcut(value = "@annotation(operationLogger)")
    public void pointcut(OperationLogger operationLogger) {
    }

    @Around(value = "pointcut(operationLogger)")
    public Object doAround(ProceedingJoinPoint joinPoint, OperationLogger operationLogger) throws Throwable {
        //先执行业务
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

### （四）使用注解

```
@OperationLogger(value = "重置用户密码")
public String restPwd() {
}
```

通过在方法上添加`@OperationLogger(value = "重置用户密码")`，就可以实现执行restPwd()方法的同时进行日志收集

## 三、注解何时生效

注解只有被解析之后才会生效，常见的解析方法有两种：

- **编译期直接扫描**：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用`@Override` 注解，编译器在编译的时候就会检测当前的方法是否重写了父类对应的方法
- **运行期通过反射处理**：像框架中自带的注解(比如 Spring 框架的`@Value` 、`@Component`)都是通过反射来进行处理的

在SpringBoot中，通过一个`@SpringBootApplication`注解包含了3个子注解

1、@SpringBootConfiguration：将Application启动类作为Bean注入到Spring容器中

2、@EnableAutoConfiguration：启动自动配置功能

3、@ComponentScan：扫描并加载组件（Component, Controller, Service, Repository等）

执行完这些流程之后就注解就生效了

## 四、注解失效

### （一）失效场景

在Spring中使用@Transactional、@Cacheable或自定义AOP注解时，会发现：在对象内部的方法中调用该对象的其他使用AOP注解的方法，被调用方法的AOP注解失效。比如

```
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

### （二）失效原因

因为不管是JDK代理还是CGLIB代理，都是通过生成一个代理类，然后在代理类中调用被代理类的方法，在调用原方法之前和之后通过一些操作来完成代理功能（比如保证事务性）。如果嵌套地调用方法，会导致方法被直接调用，而不经过代理类

### （三）解决方法

可以通过`@Autowired`自己注入自己，然后在需要嵌套调用的地方通过注入的bean调用

```
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

