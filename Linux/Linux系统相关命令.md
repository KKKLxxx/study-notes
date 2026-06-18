# Linux系统相关命令

## 一、CPU相关

### 1. top

可以查看每个进程实时的进程号、CPU占用率，内存占用率、已运行时间等信息

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202605162311411.png" alt="image-20260516231126315" style="zoom:50%;" />

## 二、内存相关

### 1. free

可以查看物理内存、交换内存的使用情况，包括总容量、已用容量、剩余容量等。配合`-h`选项更直观

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202605162319947.png" alt="image-20260516231900911" style="zoom:50%;" />

`swap`（交换空间）是磁盘上的一块区域，当内存不足时用于存放从物理内存中暂时换出的内存页（具体定义暂时不管）

### 2. vmstat

可以查看虚拟内存的使用情况

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202605162340398.png" alt="image-20260516234001366" style="zoom:50%;" />

### 3. top

top中也展示了内存信息

## 三、磁盘相关

### 1. df

可以显示每个盘的总容量、已用容量、剩余可用容量等信息，默认单位为KB，可通过`-h`选项增强可读性

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202605162340578.png" alt="image-20260516234054524" style="zoom:50%;" />

## 四、端口相关

### 1. netstat

```
netstat -nap | grep 'pid'
```

其中pid要先通过ps获取

## 五、其他

### 1. 显示并设置环境变量 export

export命令用于将shell变量输出为环境变量，或者将shell函数输出为环境变量

一个变量创建时，它不会自动地为在它之后创建的shell进程所知，而命令export可以向后面的shell传递变量的值

```
export
```

显示所有环境变量

```
export JAVA_HOME=/usr/java/jdk1.8.0_152
```

设置环境变量

### 2. 结束进程 kill

通过`kill`+进程号的方式结束一个进程，可用管道符来快速获取进程号

```
ps -ef | grep vim
root      3268  2884  0 16:21 pts/1    00:00:00 vim install.log
root      3370  2822  0 16:21 pts/0    00:00:00 grep vim

kill 3268
```

`kill`有多种选项，最常用的是[-9]（强制结束）和[-15]（正常结束），如果不加选项则默认为[-15]

对于[-15]的kill，程序会先进行结束前的准备工作，包括正常释放资源、清理临时文件等。所以这个kill信号**可能被阻塞或忽略**，导致无法立刻结束进程。这时候可以加上`-9`

```
kill -9 3268
```

然而[-9]的副作用就是可能造成一些数据的丢失

还有2个比较常用的快捷键，`CTRL + C`与`CTRL + Z`，其实它们分别相当于`kill -2`与`kill -19`

[-2]是中断，彻底结束进程

[-19]是暂停，将任务挂起，可以通过`fg`命令恢复执行

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202112041016091.png" alt="image-20211204101648981" style="zoom: 67%;" />

如图所示，启动`demo.jar`进程后，首先通过`CTRL + Z`挂起进程，之后`ps`查看进程发现该进程仍然存在。通过`fg`命令恢复执行后，再用`CTRL + C`彻底结束进程，再用`ps`查询发现进程已经不存在了
