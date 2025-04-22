# Spring Ioc

## 一、Spring Ioc定义

Ioc(Inversion of Control，控制反转)。是**一个对象定义其依赖关系而不创建它们的过程**

## 二、传统配置

```Java
public class Company {
    private Address address;
    public Company(Address address) {
        this.address = address;
    }
}

public class Address {
    private String street;
    private int number;
    public Address(String street, int number) {
        this.street = street;
        this.number = number;
    }
}
```

假设一个Company类，其包含一个Address字段，传统创建Company对象的方法是

```Java
Address address = new Address("High Street", 1000);
Company company = new Company(address);
```

想象一个有几十个甚至几百个类的应用程序。有时我们希望在整个应用程序中共享一个类的单个实例，有时我们需要为每个用例提供一个单独的对象，管理如此多的对象非常繁琐。这时就可以通过控制反转来管理

对象可以从 IoC 容器中检索其依赖项，而不是自己构建依赖项。**我们需要做的就是为容器提供适当的配置元数据**

## 三、Bean 配置

用@Component注释来装饰Company类：

```Java
@Component
public class Company {
    // this body is the same as before
}
```

这是一个向 IoC 容器提供 bean 元数据的配置类：

```Java
@Configuration
@ComponentScan(basePackageClasses = Company.class)
public class Config {
    @Bean
    public Address getAddress() {
        return new Address("High Street", 1000);
    }
}
```

配置类产生一个 Address类型的 bean 。它还带有@ComponentScan注释，它指示容器在包含Company类的包中查找 bean

**当 Spring IoC容器构造这些类型的对象时，所有对象都称为 Spring bean，因为它们由 IoC 容器管理**