# Git多人合作

当有一个开发任务时，首先要从一个远程仓库克隆，但是`git clone`之后只会克隆下来`master`分支，但我们需要在`dev`分支上开发，需要再关联一下`dev`分支，具体操作如下

```
MINGW64 /d/git测试
$ git clone git@gitee.com:KKKLxxx/test1.git
Cloning into 'test1'...
remote: Enumerating objects: 124, done.
remote: Counting objects: 100% (124/124), done.
remote: Compressing objects: 100% (103/103), done.
remote: Total 124 (delta 27), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (124/124), 14.34 KiB | 1.19 MiB/s, done.
Resolving deltas: 100% (27/27), done.

MINGW64 /d/git测试
$ cd test1

MINGW64 /d/git测试/test1 (master)
$ git branch
* master

MINGW64 /d/git测试/test1 (master)
$ git checkout -b dev origin/dev
Switched to a new branch 'dev'
Branch 'dev' set up to track remote branch 'dev' from 'origin'.

MINGW64 /d/git测试/test1 (dev)
$ git branch
* dev
  master
```

即通过`git checkout -b dev origin/dev`指令将`dev`分支也克隆下来，然后就可以开始开发了

如果一切正常，与自己单人开发是没什么差别的，就是`add`+`commit`+`push`，但是要注意一下，开发时，如果要push`dev`分支的内容，就要写作`git push origin dev`，如果是`git push origin master`，可以简写为`git push`

问题就在于有可能出现冲突，比如你和A在共同开发，当你们同时修改一个文件后，比如，A将`1.txt`中的内容修改为了

```
There is a bug
修复bug
deving
edit by A
```

并首先push到远程仓库，而你将内容修改为

```
There is a bug
修复bug
deving
edit by myself
```

经过一套`add`+`commit`+`push`后，git报错了

```
MINGW64 /d/git测试/test1 (dev)
$ git add 1.txt

MINGW64 /d/git测试/test1 (dev)
$ git commit -m 修改
[dev f9d60b4] 修改
 1 file changed, 2 insertions(+), 1 deletion(-)

MINGW64 /d/git测试/test1 (dev)
$ git push origin dev
To gitee.com:KKKLxxx/test1.git
 ! [rejected]        dev -> dev (fetch first)
error: failed to push some refs to 'gitee.com:KKKLxxx/test1.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

这时候我们要通过`git pull`来拉取远程仓库中最新的版本，然后自行解决冲突再push

```
MINGW64 /d/git测试/test1 (dev)
$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), 302 bytes | 23.00 KiB/s, done.
From gitee.com:KKKLxxx/test1
   6ae9961..930ab59  dev        -> origin/dev
Auto-merging 1.txt
CONFLICT (content): Merge conflict in 1.txt
Automatic merge failed; fix conflicts and then commit the result.
```

这时`1.txt`中的内容变为了

```
There is a bug
修复bug
deving
<<<<<<< HEAD
edit by myself
=======
edit by A
>>>>>>> 930ab597c977957f1dcb43e4a929e051ee64de5c
```

我们将冲突解决，内容修改为

```
There is a bug
修复bug
deving
edit by myself
edit by A
```

再来一套`add`+`commit`+`push`，就成功把两人的工作成功上传到远程仓库了