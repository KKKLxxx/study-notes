# B树（B-树）、B+树、红黑树

## 一、B树（B-树）

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111151957868.jpeg" alt="img" style="zoom:67%;" />

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)B树也称B-树，是一种**平衡多路查找树**。B树的定义如下

**对于一棵m阶B树**

- 每个节点最多有m-1个关键字
- 根节点最少可以只有1个关键字
- 非根节点至少有m/2个关键字
- 每个节点中的关键字都按照从小到大的顺序排列
- 所有叶子节点都位于同一层

所以，根节点的关键字数量范围：`1 `<=` k `<=` m-1`，非根节点的关键字数量范围：`m/2 `<=` k `<=` m-1`

再举个例子来说明一下上面的概念，比如一个5阶的B树，根节点数量范围：1 <= k <= 4，非根节点数量范围：2 <= k <= 4

## 二、B+树

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111151957217.png" alt="img" style="zoom:67%;" />

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

B+树其实和B树是非常相似的，我们首先看看**相同点**

- 根节点至少一个元素
- 非根节点元素范围：m/2 <= k <= m-1

**不同点**

- B+树有两种类型的节点：内部结点（也称索引结点）和叶子结点。**内部节点就是非叶子节点，内部节点不存储数据，只存储索引，数据都存储在叶子节点**
- **每个叶子结点都存有相邻叶子结点的指针**
- **父节点中的每个元素都是子节点中元素的最大或最小值**
- 内部结点中的key都按照从小到大的顺序排列

## 三、红黑树

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111151957534.jpeg" alt="img" style="zoom:67%;" />

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 红黑树是一种**特化的AVL树（平衡二叉查找树）**

红黑树的特性:
 （1）每个节点或者是黑色，或者是红色
 （2）根节点是黑色
 （3）每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点]
 （4）如果一个节点是红色的，则它的子节点必须是黑色的
 （5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点

但是上面这些特性比较晦涩难懂，总结一下来讲就是：**这些特性使得红黑树保证了相对平衡，即从根节点到叶子结点的最长路径不会大于最短路径的两倍。相比于AVL树的绝对平衡，红黑树在添加、删除时调整的代价更小，但查找的时间复杂度仍稳定保持在logn**

## 四、参考链接

[面试官问你B树和B+树，就把这篇文章丢给他](https://my.oschina.net/u/4116286/blog/3107389)