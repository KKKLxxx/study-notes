# SpringBoot打包项目为jar

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

## 方法一：直接在IDEA中打包

打开IDEA右边的工具栏，依次点击clean和package即可

![image-20211122202843572](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111222028610.png)

之后可在`pom.xml`同级目录下的`target`目录中看到打包完成的jar文件

## 方法二：命令行打包

在`pom.xml`同级目录下启动命令行，依次输入

```
mvn clean
mvn package
```

之后同上