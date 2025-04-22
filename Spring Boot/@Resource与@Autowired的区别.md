# @Resource与@Autowired的区别

两个注解均用作bean的注入

## 一、@Resource

1、JDK原生注解

2、可以根据name和type两个属性进行注入，如果有多个实现类，可以通过`name`属性或者`@Qualifier`指定

```java
@Service
public class CookTomato implements Cook {
}

@Service
public class CookPatato implements Cook {
}

// 通过name属性指定
@RestController
public class CookController {
	@Resource(name="cookTomato")
	private Cook cook;
}

// 或者通过@Qualifier指定
@RestController
public class CookController {
	@Resource
    @Qualifier("cookTomato")
	private Cook cook;
}
```

当Cook接口有两个实现类时，可以通过name属性来指定

## 二、@Autowired

1、Spring提供的注解

2、只能根据type进行注入，不会匹配name。如果有多个实现类，可以通过`@Primary`或`@Qualifier`注解指定

```java
@Service
@Primary
public class CookTomato implements Cook {
}

@Service
public class CookPatato implements Cook {
}

@RestController
public class CookController {
	@Autowired
	private Cook cook;
}
```

