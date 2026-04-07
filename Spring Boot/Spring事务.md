# Spring事务

## 一、什么是Spring事务

Spring事务是对不同底层事务的统一管理方式，它屏蔽了底层API（比如JDBC和Mybatis）的差异，使开发者可以用一致的编程模型来控制事务

（JDBC是最底层的数据库操作方式，直接写SQL；Mybatis是对JDBC的一种封装，可以将对对象执行的操作转换成SQL去执行，简化了开发）

Spring事务的特性与数据库事务的特性相同

Spring事务是否生效，与数据库引擎是否支持事务相关，比如InnoDB支持，而MyISAM则不支持

## 二、事务管理方式

### 1. 声明式事务

对方法使用`@Transactional`注解开启事务

```java
@Transactional(propagation = Propagation.REQUIRED)
public void aMethod {
    // ...
}
```

### 2. 编程式事务

通过`TransactionTemplate`手动管理事务

```java
@Autowired
private TransactionTemplate transactionTemplate;

public void testTransaction() {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
                try {
                    // 业务代码
                } catch (Exception e){
                    // 回滚
                    transactionStatus.setRollbackOnly();
                }
            }
        });
}
```

## 三、事务的传播行为

传播行为指的是，当事务方法被另一个事务方法调用时，应当开启一个新事务，还是在调用者的事务中执行

一共有7种行为：

### 1. `REQUIRED`

默认行为

- 如果当前存在事务，则加入该事务
- 如果当前没有事务，则创建一个新的事务

### 2. `REQUIRES_NEW`

总是创建新事务

### 3. `NESTED`

- 如果当前存在事务，则创建嵌套事务执行
  - 嵌套事务的特性是，子事务会随着父事务的提交或回滚而提交或回滚，不能单独提交，但可以单独回滚
- 如果当前没有事务，则创建一个新的事务

### 4. `MANDATORY`

- 如果当前存在事务，则加入该事务
- 如果当前没有事务，则抛出异常

还有三种是会以非事务的方式去执行，基本不会使用

## 四、事务的隔离级别

共5种，其中默认的是按照数据库的隔离级别去运行，剩下4种分别就是MySQL的4种

## 五、@Transactional使用方法

当`@Transactional`作用于类上时，该类的所有 public 方法将开启事务。同时，也可以在方法级别使用该注解来覆盖类级别的定义

`@Transactional`注解默认回滚策略是只有在遇到`RuntimeException`（运行时异常）或者 `Error` 时才会回滚事务，而不会回滚 `Checked Exception`（受检查异常）。这是因为 Spring 认为 `RuntimeException` 和 `Error` 是不可预期的错误，而受检异常是可预期的错误，可以通过业务逻辑来处理

如果想要修改默认的回滚策略，可以使用 `@Transactional` 注解的 `rollbackFor` 和 `noRollbackFor` 属性来指定哪些异常需要回滚，哪些异常不需要回滚。例如，如果想要让所有的异常都回滚事务，可以使用如下的注解：

```java
@Transactional(rollbackFor = Exception.class)
public void someMethod() {

}
```

如果想要让某些特定的异常不回滚事务，可以使用如下的注解：

```java
@Transactional(noRollbackFor = CustomException.class)
public void someMethod() {

}
```

