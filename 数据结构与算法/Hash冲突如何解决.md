# Hash冲突如何解决

1、**链地址法**：对哈希值冲突的结点用一个链表/红黑树连接（Java中的HashMap就是用这种方法解决的）

2、**开放定址法**：一直往后直到找到一个空的位置

3、**再哈希法**：再次计算哈希值

4、**公共溢出区**：用一个公共地址保存冲突元素

