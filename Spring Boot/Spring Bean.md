# Spring Bean

## 一、Spring Bean定义

Bean 是由 Spring IoC 容器创建和管理的对象

## 二、作用域

1、**单例**：IoC 容器中只有唯一的 bean 实例。Spring 中的 bean **默认**都是单例的，是对单例设计模式的应用

2、**原型**：每次获取都会创建一个新的 bean 实例

3、**request**：每次HTTP请求都创建一个新的实例，仅适用于Web应用

4、**session**：同一个HTTP session共享一个实例，仅适用于Web应用

5、**global-session**：④的特殊版，仅适用于基于Portlet的Web应用

## 三、生命周期

创建 -> PostConstruct -> 使用 -> PreDestory ->销毁

**对于原型Bean**：由IoC容器创建完成后，其生命周期交给程序员管理，对Bean的销毁需要由程序员手动执行，所以不会自动执行PreDestory方法

具体指定方法是在bean内通过`@PostConstruct`和`@PreDestory`注解指定

```java
@PostConstruct
private void postConstruct() {
    ...
}

@PreDestroy
public void preDestroy() {
    ...
}
```

## 四、Bean 是线程安全的吗？

Bean 是否线程安全，取决于其作用域和状态（也就是有无成员变量）

prototype 作用域下，每次获取都会创建一个新的 bean 实例，不存在资源竞争问题，所以不存在线程安全问题

singleton 作用域下，如果这个 bean 是有状态的话，那就存在线程安全问题。不过大部分 Bean 实际都是无状态的（比如 Dao、Service），这种情况下 Bean 是线程安全的

对于有状态单例 Bean 的线程安全问题，常见的解决办法是：

- 尽量将 Bean 设计为无状态
- 使用ThreadLocal存储变量，保证线程独立
- 使用synchronized或Reentrantlock等同步机制

## 五、将一个类或方法声明为 Bean 的注解有哪些？

- `@Component`：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注
- `@Bean`：方法级别的注解，通过手动编写创建逻辑注册 Bean，适用于第三方类或需要精细控制的场景
- `@Configuration`：用于集中定义 Bean 的配置类

- `@Repository`：对应持久层即 Dao 层，主要用于数据库相关操作

- `@Service`：对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层

- `@Controller`：对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面

后四种内部都标注了 `@Component`，属于 `@Component` 的特化形式，用于更清晰地表达分层语义，但功能完全相同

## 六、@Component与@Bean的区别

@Component修饰类，@Bean修饰方法

当需要自定义的类作为Bean时，使用@Component或者其派生注解

当需要第三方库中的类作为Bean时（如 DataSource、RestTemplate），由于无法修改其源码在其类上加上@Component，所以要使用@Bean方法显式创建并返回。@Bean要与@Configuration配合使用，从而保证@Bean方法可以被扫描到

当 Bean 的创建需要复杂的初始化逻辑时（如读取配置文件、条件判断、多个步骤构建），也可以通过@Bean方法来定义

```java
// 自定义类，使用 @Component
@Component
public class MyService {
    public void doWork() { }
}

// 配置类，使用 @Bean 创建第三方对象
@Configuration
public class MyConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

## 七、注入 Bean 的注解有哪些？

### 1. @Resource— Java 规范方式

@Resource用于按照**名称**自动注入 Bean，如果存在多个，则按照类型进行匹配

```java
@Resource
private UserService userService;

@Resource(name = "userServiceImpl")		// name字段可选
private UserService userService;
```

### 2. @Autowired—Spring 推荐方式

@Autowired用于按照**类型**自动注入 Bean，如果存在多个，则按照名称进行匹配（可以配合@Qualifier指定）

```java
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
```

### 3. 对比

更推荐使用@Autowired，因为它是Spring提供的，更适合Spring架构的程序，并且先按类型匹配的逻辑更符合面向接口编程的思想

面向接口编程：使用接口（或抽象类型）来定义模块之间的交互，而非直接依赖具体的实现类，从而降低耦合、易于扩展

## 八、注解何时生效

注解只有被解析之后才会生效，常见的解析方法有两种：

- **编译期直接扫描**：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用`@Override` 注解，编译器在编译的时候就会检测当前的方法是否重写了父类对应的方法
- **运行期通过反射处理**：像框架中自带的注解（比如 Spring 框架的`@Value` 、`@Component`）都是通过反射来进行处理的

在SpringBoot中，通过一个`@SpringBootApplication`注解包含了3个子注解

1、@SpringBootConfiguration：将Application启动类作为Bean注入到Spring容器中

2、@EnableAutoConfiguration：启动自动配置功能

3、@ComponentScan：扫描并加载组件（Component, Controller, Service, Repository等）

执行完这些流程之后就注解就生效了

## 九、注解失效

### 1. 失效场景

在Spring中使用@Transactional、@Cacheable或自定义AOP注解时，会发现：**在对象内部，直接调用该对象其他使用AOP注解的方法，会使被调用的方法注解失效**。比如

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

### 2. 失效原因

因为不管是JDK代理还是CGLIB代理，都是通过生成一个代理类，然后在代理类中调用被代理类的方法。如果嵌套地调用方法，会导致方法被直接调用，而不经过代理类

### 3. 解决方法

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