# Spring Bean作用域与生命周期

## 一、Spring Bean定义

构成应用程序主干并由 Spring IoC 容器管理的对象称为 bean。bean 是由 Spring IoC 容器实例化、组装和管理的对象

## 二、作用域

1、**单例**：适用于无状态Bean（因为线程安全，无状态即无实例字段），**默认**

2、**原型**：每次调用都返回一个新的实例，适用于有状态Bean

3、**request**：每次HTTP请求都创建一个新的实例，仅适用于Web应用

4、**session**：同一个HTTP session共享一个实例，仅适用于Web应用

5、**global-session**：（4）的特殊版，仅适用于基于Portlet的Web应用

## 三、生命周期

实例化Bean -> PostConstruct -> 通过实现一些接口执行某些方法 -> PreDestory ->销毁

**原型**：创建完成后，对Bean的销毁需要由程序员管理，所以不会自动执行PreDestory方法

具体指定方法是在bean内通过`@PostConstruct`和`@PreDestory`注解指定

```Plain
@PostConstruct
private void postConstruct() {
    ...
}

@PreDestroy
public void preDestroy() {
    ...
}
```