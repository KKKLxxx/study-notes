# Xshell中将Linux与Windows中的文件互传

## 一、安装

安装lrzsz工具包

```
yum -y install lrzsz
```

## 二、使用

### (一)、Windows -> Linux

#### 1、rz命令

在Linux命令行输入rz后，就会打开一个Windows的文件选择框，选中文件后便可以将文件传到Linux的当前目录下

#### 2、直接拖拽

可以直接缩小窗口后将文件拖入Xshell中，实现复制

### (二)、Linux -> Windows

只能通过`sz filename`指令传输

比如要传输1.txt这个文件，就输入`sz 1.txt`，然后又会打开一个Windows的文件选择框，选择好目录后确认即可

### (三)、注意

不能传输文件夹，但可以将文件夹压缩后再传输

## 三、Ubuntu下安装

上面是CentOS版本的，而Ubuntu下可能没有yum，但是可以通过如下命令安装，使用同上

```
apt-get install lrzsz
```

