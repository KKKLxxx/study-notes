# Spring相关问题

## 一、Spring MVC

MVC，Model View Controller（模型-视图-控制器），是一种设计模式，用一种数据、界面、业务逻辑分离的方式组织代码，从而降低代码的耦合

- **模型**：代表一个存储数据的对象，主要用于承载数据，还可对用户提交的请求进行计算，模型分为两类：
  - 数据承载 Bean：指实体类（如：User类），专门为用户承载业务数据
  - 业务处理 Bean：指 Service 或 Dao 对象， 专门用于处理用户提交的请求
- **视图**：为用户提供使用界面，与用户直接进行交互
- **控制器**：用于将用户请求转发给相应的 Model 进行处理，并根据 Model 的计算结果向用户提供相应响应，它使视图与模型分离

## 二、Servlet与Servlet容器

### 1. Servlet

Web应用通过HTTP来进行请求，HTTP服务器的作用主要是处理TCP连接并解析HTTP协议。主要流程如下

- 识别正确和错误的HTTP请求
- 识别正确和错误的HTTP头
- 复用TCP连接
- 复用线程
- IO异常处理
- ...

Servlet是一种专用于Java平台的底层网络连接处理技术，它可以使程序员更加关注于业务本身。Servlet是平台独立的Java类，编写一个Servlet，实际上就是按照Servlet规范编写一个Java类。Servlet被编译为平台独立的字节码，可以被动态地加载到支持Java技术的Web服务器中运行

### 2. Servlet容器

Servlet容器是Web服务器或应用程序服务器的一部分。Servlet没有main方法，不能独立运行，它必须被部署到Servlet容器中，由容器来实例化和调用Servlet的方法（如doGet()和doPost()），Servlet容器在Servlet的生命周期内管理Servlet

常见的Servlet容器：Tomcat

### 3. DispatcherServlet

DispatcherServlet 是 Spring MVC 的核心控制器，它接收所有请求，然后分发给对应的 Controller 处理

## 三、过滤器与拦截器的区别

### 1. 过滤器

过滤器来自 Servlet 规范，通过实现`javax.servlet.Filter`接口来实现

作用：在请求进入 Servlet 之前和响应返回客户端之前进行处理

示例代码：

```java
@Component
public class LogFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request,
                         ServletResponse response,
                         FilterChain chain) {
        System.out.println("过滤器执行");
        chain.doFilter(request, response);
    }
}
```

### 2. 拦截器

拦截器来自 Spring MVC，通过实现`HandlerInterceptor`接口来实现

作用：在 Controller 方法执行前后进行增强

示例代码：

```java
@Component
public class LogInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) {
        System.out.println("拦截器执行");
        return true;
    }
}
```

注册：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor());
    }
}
```

### 3. 对比

| 对比点                     | 过滤器                 | 拦截器     |
| -------------------------- | ---------------------- | ---------- |
| 所属规范                   | Servlet                | Spring MVC |
| 执行顺序                   | 先执行                 | 后执行     |
| 是否依赖 Spring            | 否                     | 是         |
| 是否能拿到 Controller 方法 | 否                     | 可以       |
| 是否可修改 request         | 可以                   | 可以       |
| 是否可获取 Spring Bean     | 默认不行（需特殊处理） | 可以       |

一个 HTTP 请求进入后的顺序：

```
客户端
   ↓
Servlet容器
   ↓
Filter（过滤器）
   ↓
DispatcherServlet
   ↓
Interceptor（拦截器 preHandle）
   ↓
Controller
   ↓
Interceptor（postHandle）
   ↓
视图渲染
   ↓
Interceptor（afterCompletion）
   ↓
Filter（响应返回阶段）
   ↓
客户端
```

过滤器是基于 Servlet 规范的组件，在请求进入 DispatcherServlet 之前执行，可以对所有请求进行过滤

拦截器是 Spring MVC 提供的机制，在 Controller 方法执行前后进行增强

过滤器更底层，用于控制“通用的请求流”，比如修改 request / response、跨域处理

拦截器更接近业务层，用于控制“具体的业务方法”，比如登录校验、权限控制、日志记录、统计接口耗时

## 四、SpringBoot相关问题

### 1. SpringBoot比Spring好在哪？

Spring Boot 是对 Spring 的增强，主要是通过自动装配机制减少配置工作（比如对Servlet容器的配置），并通过`spring-boot-starter`简化依赖管理，提高开发效率

### 2. SpringBoot自动装配原理

Spring Boot 启动时通过 @EnableAutoConfiguration 加载自动配置类，并通过条件注解决定是否注册 Bean（比如`@ConditionalOnClass(DispatcherServlet.class)`表示只有当类路径中存在 DispatcherServlet 才生效）

## 五、@Value、@Resource、@Autowired的区别

### 1. @Value

@Value用于注入值，常用于读取 application.properties

示例：

```java
@Value("${server.port}")
private int port;

@Value("hello")
private String name;

@Value("#{2 * 3}")
private int result;
```

### 2. @Resource— Java 规范方式

@Resource用于按照**名称**自动注入 Bean，如果存在多个，则按照类型进行匹配

```java
@Resource
private UserService userService;

@Resource(name = "userServiceImpl")		// name字段可选
private UserService userService;
```

### 3. @Autowired—Spring 推荐方式

@Autowired用于按照**类型**自动注入 Bean，如果存在多个，则按照名称进行匹配（可以配合@Qualifier指定）

```java
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
```

### 4. 对比

@Value用于注入值，@Resource与@Autowired用于注入Bean

更推荐使用@Autowired，因为它是Spring提供的，更适合Spring架构的程序，并且先按类型匹配的逻辑更符合面向接口编程的思想

## 六、程序打包与运行

### 1. 打包

首先确保在`pom.xml`中引入了maven依赖

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

#### 1.1 方法一：直接在IDEA中打包

打开IDEA右边的工具栏，依次点击clean和package即可

![image-20211122202843572](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111222028610.png)

之后可在`pom.xml`同级目录下的`target`目录中看到打包完成的jar文件

#### 1.2 方法二：命令行打包

在`pom.xml`同级目录下启动命令行，依次输入

```
mvn clean
mvn package
```

之后同上

### 2. 运行

#### 2.1 前台运行

```
java -jar demo.jar
```

直接运行以上命令，会直接在终端显示启动内容，并且输入`^C`后会直接结束程序

#### 2.2 半后台运行

```
java -jar demo.jar &
```

再末尾加上`&`之后，它仍会跳转到程序输出界面，但可以输入`^C`退出该界面，并且程序不会结束。但是如果关闭终端窗口/断开SSH连接，就会结束程序

#### 2.3 完全后台运行

```
nohup java -jar demo.jar &
```

这样的话即使关闭窗口也不会结束程序

不过这时候有一个提示

```
nohup: ignoring input and appending output to ‘nohup.out’
```

这里说明会将程序的输出内容重定向到默认的`nohup.out`文件中，我们也可以指定日志文件

```
nohup java -jar demo.jar > log.txt &
```

此时又会提示

```
nohup: ignoring input and redirecting stderr to stdout
```

这里可以通过添加一个`2>&1`解决，参考https://blog.csdn.net/zhaominpro/article/details/82630528

```
nohup java -jar demo.jar > log.txt 2>&1 &
```

#### 2.4 结束进程

```
[root@iZtwa2bngfuo32Z ~]# ps -ef | grep demo
root     10213  9039  2 23:30 pts/0    00:00:07 java -jar demo.jar
root     10523  9039  0 23:35 pts/0    00:00:00 grep --color=auto demo
[root@iZtwa2bngfuo32Z ~]# kill 10213
```

