# Servlet与Servlet容器

## 一、Servlet

Web应用通过HTTP来进行请求，HTTP服务器的作用主要是处理TCP连接并解析HTTP协议。主要流程如下

- 识别正确和错误的HTTP请求
- 识别正确和错误的HTTP头
- 复用TCP连接
- 复用线程
- IO异常处理
- ...

Servlet是一种专用于Java平台的底层网络连接处理技术，它可以使程序员更加关注于业务本身。Servlet是平台独立的Java类，编写一个Servlet，实际上就是按照Servlet规范编写一个Java类。Servlet被编译为平台独立的字节码，可以被动态地加载到支持Java技术的Web服务器中运行

## 二、Servlet容器

Servlet容器是Web服务器或应用程序服务器的一部分。Servlet没有main方法，不能独立运行，它必须被部署到Servlet容器中，由容器来实例化和调用Servlet的方法（如doGet()和doPost()），Servlet容器在Servlet的生命周期内管理Servlet

常见的Servlet容器：Tomcat

