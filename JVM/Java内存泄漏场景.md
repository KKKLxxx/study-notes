# Java内存泄漏场景

## 一、静态集合类

如果不正确的使用静态集合类，那么加到集合中的元素生命周期就会被延长，可能导致内存泄漏

## 二、没有重写equals与hashCode方法

当用自定义类作为HashMap的key时，如果没有重写这两个方法，也可能使内容相同的对象在HashMap中重复出现，导致内存泄漏

## 三、未关闭资源

比如文件读写、数据库连接、网络连接

可以通过try-catch-finally或try-with-resource语句保证资源关闭

## 四、ThreadLocal没有调用remove方法

虽然ThreadLocal可以自动清理，但是最好在使用完后自行调用remove方法及时清理

