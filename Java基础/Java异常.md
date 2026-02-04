# Java异常

## 一、异常类

在Java中，所有的异常都有一个共同的祖先`java.lang`包中的`Throwable`类。`Throwable`类有两个重要的子类

### 1. Error

准确来讲是错误，常见的有OutOfMemoryError（内存耗尽）和StackOverflowError（栈溢出）等，这类错误一般是因为代码本身的问题

### 2. Exception

这个才是通常意义上的异常，可以由try-catch语法捕获的那种。Exception又可以分为Checked Exception（受检查异常，必须处理）和Unchecked Exception（不受检查异常，可以不处理）

#### 2.1 Checked Exception

**Checked Exception**即**受检查异常**，在编译过程中，如果受检查异常没有被`catch`或`throws`关键字处理的话，就没办法通过编译

比如下面这段 IO 操作的代码：

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6769746875622f6a61766167756964652f6a6176612f62617369732f636865636b65642d657863657074696f6e2e706e67)

除了`RuntimeException`及其子类以外，其他的`Exception`类及其子类都属于受检查异常。常见的受检查异常有：IO相关异常`IOException`、`FileNotFoundException` 、`SQLException`

#### 2.2 Unchecked Exception

**Unchecked Exception**即**不受检查异常**，在编译过程中 ，即使不处理不受检查异常也可以正常通过编译

`RuntimeException` 及其子类都统称为非受检查异常，常见的有：

- `NullPointerException`（空指针错误）

- `ArrayIndexOutOfBoundsException`（数组越界错误）

- `ArithmeticException`（算术错误）

## 二、异常处理

### 1. try-catch-finally

- `try`块：用于捕获异常。其后可接零个或多个 `catch` 块，如果没有 `catch` 块，则必须跟一个 `finally` 块
- `catch`块：用于处理 try 捕获到的异常
- `finally`块：无论是否捕获或处理异常，`finally` 块里的语句都会被执行。当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行（**如果`finally`中有return语句，那么就会返回`finally`中的返回值**）

代码示例：

```java
try {
    System.out.println("Try to do something");
    throw new RuntimeException("RuntimeException");
} catch (Exception e) {
    System.out.println("Catch Exception -> " + e.getMessage());
} finally {
    System.out.println("Finally");
}
```

输出：

```
Try to do something
Catch Exception -> RuntimeException
Finally
```

### 2. try-with-resources

《Effective Java》中指出：

> 面对必须要关闭的资源，我们总是应该优先使用`try-with-resources`而不是`try-catch-finally`。随之产生的代码更简短，更清晰，产生的异常对我们也更有用。`try-with-resources`语句让我们更容易编写必须要关闭的资源的代码，若采用`try-catch-finally`则几乎做不到这点

1、**适用范围（资源的定义）**：任何实现`java.lang.AutoCloseable`或者`java.io.Closeable`的类

2、**关闭资源和finally块的执行顺序**：在`try-with-resources`语句中，任何 catch 或 finally 块在声明的资源关闭后运行

Java中类似于`InputStream`、`OutputStream` 、`Scanner` 、`PrintWriter`等的资源都需要我们调用`close()`方法来手动关闭，一般情况下我们都是通过`try-catch-finally`语句来实现这个需求，如下：

```java
// 读取文本文件的内容
Scanner scanner = null;
try {
    scanner = new Scanner(new File("D://read.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

使用Java 7之后的 `try-with-resources` 语句改造上面的代码：

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

当然多个资源需要关闭的时候，使用`try-with-resources`实现起来也非常简单，如果你还是用`try-catch-finally`可能会带来很多问题

通过使用分号分隔，可以在`try-with-resources`块中声明多个资源

```java
try (BufferedInputStream bin = new BufferedInputStream(new FileInputStream(new File("test.txt")));
     BufferedOutputStream bout = new BufferedOutputStream(new FileOutputStream(new File("out.txt")))) {
    int b;
    while ((b = bin.read()) != -1) {
        bout.write(b);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

## 三、throw与throws

### 1. throw

用于手动抛出异常。可以根据需要在代码中使用throw语句主动抛出特定类型的异常

```java
throw new ExceptionType("Exception message");
```

### 2. throws

用于在方法声明中声明可能抛出的异常类型。如果一个方法可能抛出异常，但不想在方法内部进行处理，可以使用throws关键字将异常传递给调用者来处理

```java
public void methodName() throws ExceptionType {

}
```

