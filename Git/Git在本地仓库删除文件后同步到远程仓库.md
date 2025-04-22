# Git在本地仓库删除文件后同步到远程仓库

## 一、删除本地文件

```
git rm 1.txt
```

删除本地文件`1.txt`，可通过`-r`选项删除文件夹

`git rm`指令可以直接删除本地文件，并让git知道。也可以先自行删除，再`git rm`

## 二、提交修改

```
git commit -m 删除文件
```

## 三、同步远程仓库

```
git push origin master
```

