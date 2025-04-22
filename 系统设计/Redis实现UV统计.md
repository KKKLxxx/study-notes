# Redis实现UV统计

## 一、UV的概念

**U**nique **V**isitor，也叫独立访问量，指的是一天内访问某网站的用户数量，若一个用户在一天内访问多次，则只记录一次

## 二、常规实现

常规的实现方式，就是创建一个与日期相关的key，并且这个key是set类型，value为每个用户的ID

这种方式的缺陷在于，当UV非常大时，每天都会产生一个大key，非常占用内存

## 三、HyperLogLog

Redis HyperLogLog是用来做基数统计的算法，HyperLogLog的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的，并且是很小的

在Redis中，每个HyperLogLog键只需要花费小于16KB的内存，就可以计算接近2^64个不同元素的基数

但是因为HyperLogLog只会根据输入元素来计算基数，而不会储存输入元素本身，所以HyperLogLog不能像集合那样，返回输入的各个元素。并且HyperLogLog存在小于0.81%的误差，但是对于UV统计来说，这个误差可以忽略不计

## 四、相关指令

1、添加指定元素到 HyperLogLog 中

`PFADD key element [element ...]`

2、返回给定 HyperLogLog 的基数估算值

`	PFCOUNT key [key ...]`

3、将多个 HyperLogLog 合并为一个 HyperLogLog

`PFMERGE destkey sourcekey [sourcekey ...]`