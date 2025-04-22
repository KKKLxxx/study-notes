# Linux中tar格式的压缩与解压

## 一、压缩

```
tar -czvf 1.tar.gz 1.txt
```

将当前目录下的`1.txt`文件压缩为`1.tar.gz`

## 二、解压

```
tar -xzvf 1.tar.gz
```

将当前目录下`1.tar.gz`解压到当前文件夹

## 三、查看压缩文件

```
tar -tzvf 1.tar.gz
```

列出压缩文件`1.tar.gz`中的内容

