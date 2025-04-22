# Buddy分配与slab分配

## 一、Buddy分配

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/6829fFuXmMJKeaY.png" alt="img" style="zoom:50%;" />

伙伴系统进行2的幂次大小的分配。当有一个21KB大小的请求时，伙伴系统会从256KB一直分裂为32KB，然后进行分配

### 缺点：

内存碎片，极端情况可造成接近50%的浪费

## 二、slab分配

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/Hb81ZtRaynSOvNG.png" alt="1280X1280" style="zoom:50%;" />

slab系统分为3部分：内核对象object、高速缓存cache、slab

每个slab由一个或多个物理连续的页面组成，每个cache由一个或多个slab组成，每个object对应一个cache

slab系统为内核对象预先分配好相同大小的slab，当请求到来时，cache利用已经在slab中分配的并且标记为free的对象来满足请求

### 优点：

1、减少内存碎片

2、分配速度快。由于对象预先创建，所以省略了内存分配的时间。当内存释放时，可以直接把cache标记为空闲，进行复用（类似线程池）