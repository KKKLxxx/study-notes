# 后台启动jar

## 1、前台运行

```
java -jar demo.jar
```

直接运行以上命令，会直接在终端显示启动内容，并且输入`^C`后会直接结束程序

## 2、半后台运行

```
java -jar demo.jar &
```

再末尾加上`&`之后，它仍会跳转到程序输出界面，但可以输入`^C`退出该界面，并且程序不会结束。但是如果关闭终端窗口/断开SSH连接，就会结束程序

## 3、完全后台运行

```
nohup java -jar demo.jar &
```

这样的话即使关闭窗口也不会结束程序

不过这时候有一个提示

```
nohup: ignoring input and appending output to ‘nohup.out’
```

这里说明会将程序的输出内容重定向到默认的`nohup.out`文件中，我们也可以指定日志文件

```
nohup java -jar demo.jar > log.txt &
```

此时又会提示

```
nohup: ignoring input and redirecting stderr to stdout
```

这里可以通过添加一个`2>&1`解决，参考https://blog.csdn.net/zhaominpro/article/details/82630528

```
nohup java -jar demo.jar > log.txt 2>&1 &
```

## 4、结束进程

```
[root@iZtwa2bngfuo32Z ~]# ps -ef | grep demo
root     10213  9039  2 23:30 pts/0    00:00:07 java -jar demo.jar
root     10523  9039  0 23:35 pts/0    00:00:00 grep --color=auto demo
[root@iZtwa2bngfuo32Z ~]# kill 10213
```

