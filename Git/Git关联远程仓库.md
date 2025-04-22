# Git关联远程仓库

## 一、本地文件夹关联远程仓库

### 1、创建本地文件夹

### 2、初始化git

进入文件夹后，通过

```
git init
```

指令初始化

### 3、提交已有文件

通过

```
git add README.md
```

指令，将所有已有文件add

### 4、提交

```
git commit -m "首次提交"
```

### 5、创建远程仓库

直接在Gitee上创建即可

### 6、关联远程仓库

```
git remote add origin git@gitee.com:KKKLxxx/study-notes.git
```

origin会成为远程仓库的名字，可以自行修改

### 7、推送到远程仓库

```
git push -u origin master
```

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令，不加参数-u



## 二、克隆远程仓库

### 1、在本地克隆

```
git clone git@gitee.com:KKKLxxx/test1.git
```

然后就成功了，之后的操作就是add、commit、push...



## 三、查看/删除远程仓库

如果添加的时候地址写错了，或者就是想删除远程库，可以用`git remote rm <name>`命令。使用前，建议先用`git remote -v`查看远程库信息：

```
$ git remote -v
origin  git@gitee.com:KKKLxxx/test1.git (fetch)
origin  git@gitee.com:KKKLxxx/test1.git (push)
```

然后，根据名字删除，比如删除`origin`：

```
$ git remote rm origin
```

此处的“删除”其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库。远程库本身并没有任何改动。要真正删除远程库，需要登录到Gitee，在后台页面找到删除按钮再删除。



## 四、注意

**空文件夹上传后，在远程仓库中并不会显示**，但会保留。当用户在空文件夹内上传了文件并推送后，远程仓库中会正常显示有文件的文件夹