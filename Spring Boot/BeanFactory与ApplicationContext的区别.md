# BeanFactory与ApplicationContext的区别

BeanFactory与ApplicationContext都可以作为Spring容器，用于管理Spring bean

## 一、BeanFactory

能够实现基本的获取bean信息、加载bean、管理bean的生命周期等功能

加载方式：**懒汉式**

## 二、ApplicationContext

ApplicationContext继承于BeanFactory，在BeanFactory的基础上还能支持国际化等功能

加载方式：**饿汉式**