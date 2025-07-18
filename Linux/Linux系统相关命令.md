# Linux系统相关命令

### 显示并设置环境变量 export

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

### 磁盘使用情况 df

可以显示每个盘的总容量、已用容量、剩余可用容量等信息，默认单位为KB，可通过`-h`选项增强可读性

### 查看系统整体运行情况 top

直接输入`top`指令，可以查看每个进程实时的进程号、CPU占用率，内存占用率、已运行时间等信息

### 显示当前进程的状态 ps

可以查看当前时刻的进程状态，有很多选项可以搭配。与top的区别主要在于，top是持续监测，ps是一个时刻的状态；top可以查看CPU、内存等资源的使用情况，ps不可以

### 查看进程端口号 netstat

```
netstat -nap | grep 'pid'
```
其中pid要先通过ps获取

### 显示内存使用情况 free

可以查看物理内存、虚拟内存的使用情况，包括总容量、已用容量、剩余容量等。配合`-h`选项更直观

### 显示虚拟内存状态 vmstat

### 结束进程 kill

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
