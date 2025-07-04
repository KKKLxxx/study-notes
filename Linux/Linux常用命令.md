# Linux常用命令

### 显示当前目录绝对路径 pwd

```
pwd
```

### 显示目录中的文件 ls

```
ls
```

| 选项 | 作用                                 |
| ---- | ------------------------------------ |
| -a   | 显示所有文件，包括隐藏文件           |
| -l   | 以列表形式显示详细信息，可缩写为`ll` |

### 目录跳转 cd

Linux中，`/`代表根目录，`~`代表`/root`目录，在远程连接时，默认会在`/root`目录下。`.`表示当前目录，`..`表示上级目录

```
cd
cd ~
```

这两条指令都表示跳转到`/root`下

```
cd ..
```

跳转到上级目录

```
cd -
```

跳转到前一个目录

```
cd 绝对路径
```

通过写上绝对路径可以直接指定跳转位置

### 删除 rm

```
rm
```

| 选项 | 作用                       |
| ---- | -------------------------- |
| -f   | 强制删除                   |
| -r   | 递归地删除，用于删除文件夹 |
| -i   | 在删除每个文件前询问       |

### 移动或重命名 mv

```
mv source target
```

source表示源文件或目录，target表示目标文件或目录。如果将一个文件移到一个已经存在的目标文件中，则**目标文件的内容将被覆盖**

```
mv 1.txt 2.txt
```

将文件`1.txt`重命名为`2.txt`

```
mv 1.txt folder
```

将文件`1.txt`重命名为`folder`

```
mv 1.txt folder/
```

将文件`1.txt`移动到本目录下的folder文件夹下，注意与上一条指令的区别

```
mv 1.txt folder/2.txt
```

将文件`1.txt`移动到本目录下的folder文件夹下，并重命名为`2.txt`

```
mv folder2/* folder/
```

将文件夹`folder2`中的所有文件移动到本目录下`folder`文件夹中

```
mv folder2 folder/
```

将文件夹`folder2`移动到本目录下`folder`文件夹中

### 复制 cp

用法是与`mv`类似的

```
cp 1.txt 2.txt
```

将文件`1.txt`复制并将复制后的文件命名为`2.txt`

```
cp -r folder2 folder
cp -r folder2 folder/
```

将文件夹`folder2`复制到`folder`文件夹下，注意**复制文件夹要加`-r`，这里与`mv`不同**

### 创建空目录 mkdir

```
mkdir folder
```

创建名为`folder`的空目录

### 删除空目录 rmdir

```
rmdir folder
```

删除名为`folder`的空目录，注意只能删除**空目录**，如果`folder`中有其他文件，可以使用`rm -r folder`快速删除，但是有一定误删风险

### 创建空文件 touch

```
touch 1.txt
```

快速创建一个名为`1.txt`的空文件

### 显示文件内容 cat

```
cat 1.txt
```

显示`1.txt`的内容到控制台上

### 搜索文本并打印 grep

`grep`使用正则表达式搜索文本，并把匹配的行打印出来

```
grep 1 1.txt
```

在`1.txt`中搜索所有**包含**`1`的行并打印

```
grep -v 1 1.txt
```

在`1.txt`中搜索所有**不**包含`1`的行并打印

```
grep 1 1.txt 2.txt
```

在`1.txt`和`2.txt`中搜索所有包含`1`的行并打印

```
grep -c 1 1.txt
```

统计并打印`1.txt`中包含`1`的行的行数

```
grep -i A 1.txt
```

在`1.txt`中搜索所有包含`A`或`a`的行并打印（`-i`说明忽略大小写）

```
grep 1 1.txt -A 3
```

`-A`表示打印匹配行及其后n行，同理，`-B`表示打印匹配行及其前n行，`-C`表示打印匹配行及其前后各n行

```
ps -ef | grep vim
```

常与管道符连用，快速获取进程号后可结束进程

### 软件包管理 yum

yum是基于rpm的软件包管理器，它能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装

```
yum -y install 软件包名
```

`-y`表示对所有问题回答`yes`

### 在指定目录下查找文件 find

```
find /home -name "*.txt"
```

即查找在`home`目录**及其子目录**下，查找所有以`.txt`为后缀的文件。可以用`.`表示当前目录

```
find /home -iname "*.txt"
```

在上一条命令的基础上忽略大小写

```
find . ! -name "*.txt"
```

查找当前目录下不以`.txt`结尾的所有文件

```
find . -name "*.txt" -o -name "*.pdf"
```

通过`-o`连接，查找当前目录下所有以`.txt`和`.pdf`结尾的文件

### 通过程序名查找 whereis

whereis命令**只能用于程序名的搜索**，whereis的速度快于find，因为find是遍历磁盘搜索，而whereis会遍历一个系统自建的数据库，但有数据更新不及时的风险

```
whereis nginx
whereis -b java
```

查找`nginx`和`java`的二进制文件所在的位置，可加参数`-b`只查找二进制文件

### 文件查找的另一种方式 locate

locate的作用于find相同，但查找方法与whereis类似，也是在一个数据库中查找，也有数据更新不及时的缺点

使用前可能需要自行安装

```
yum -y install mlocate
```

在使用前最好通过`updatedb`更新一次数据库

```
locate /etc/sh
```

搜索etc目录下所有以sh开头的文件

```
locate -i ~/m
```

搜索用户主目录下，所有以m开头的文件，并且忽略大小写

