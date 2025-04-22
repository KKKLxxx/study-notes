# Git版本回退

假设当前有1个文件`1.txt`，内容为`1`。然后我们将其内容修改为`2`并保存后（**此时还未add**），想要退回到内容为`1`的那个版本

此时可以先用`git status`查看一下距上一次`commit`后有哪些变动

```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   1.txt
```

可以看到`modified:   1.txt`，说明`1.txt`被修改了

git提示我们

```
(use "git restore <file>..." to discard changes in working directory)
```

可以用`git restore <file>`指令丢弃工作区的修改，那么就直接输入以下指令修改即可

```
git restore 1.txt
```

这样就回退成功了

---

**如果此时已经`add`了要如何回退？**

这时就不能向上面一样直接`git resrore`了，继续使用`git status`查看修改，会有如下提示

```
(use "git restore --staged <file>..." to unstage)
```

意思是，可以用这个指令撤销在暂存区的提交，那么就执行

```
git restore --staged 1.txt
```

这时候`1.txt`中的内容不会变化，但是git的状态已经回到提交前了，就可以直接使用`git restore 1.txt`回退了

---

**一种比较直接的方式**

上面演示`add`后想回退还要先撤销暂存区，再撤销修改，略显麻烦，可以通过`git log`获取想要回退的版本号

```
$ git log
commit 514c945c6ef2812d69324e380b505df857feebb6 (HEAD -> master, origin/master, origin/HEAD)
Author: KKKLxxx <1223344454@qq.com>
Date:   Wed Nov 3 14:39:10 2021 +0800

    1
```

在git中，`HEAD`表示当前版本，`HEAD^`表示上一个版本，`HEAD^^`表示上上个版本，以此类推。如果`^`太多的话，容易出错，可以用直接使用版本号，版本号也不需要全部输入，取前几位即可

版本号会在`commit`之后才更新，由于将内容由`1`修改为`2`后没有提交，所以此时的`HEAD`指向内容为`1`的版本，可以通过以下指令回退

```
git reset --hard HEAD
```

也可以把`HEAD`改为版本号

```
git reset --hard 514c
```

这样就成功回退了，这样**暂存区的内容也会被清空**

---

**如果此时已经`commit`了要如何回退？**

其实`commit`之前之后的区别就在于改变了`HEAD`的指针，如果想通过`HEAD`来回退到上一个版本，就不能像上面一样写`HEAD`了，而要改为

```
git reset --hard HEAD^
```

当然，如果是通过版本号回退，那么指令是没有变化的

---

**如果此时已经`push`了要如何回退？**

（以我现在的理解）这种情况只能在本地回退，无法回退远程仓库。如果输入`git push`，会提示本地提交落后于远程仓库的提交，让你通过`git pull`获取远程仓库最新的版本再自行修改

---

**如果回退之后想复原怎么办？**

假设我们修改内容为`2`后，通过回退功能退回了内容为`1`的版本，但是此时又想回到内容为`2`的版本

此时可以通过`git log`查看一下版本号，然后就会发现上一次提交的内容已经没有了，但是我们仍然可以通过`git reflog`指令查看“被回退”的版本

```
$ git reflog
514c945 (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
c03cf53 (origin/master, origin/HEAD) HEAD@{1}: commit: 内容修改为2
```

现在重新获得了版本号，再通过`reset`指令回退即可

```
git reset --hard c03c
```

如果命令行没有关闭，而且之前查询过版本号，即可直接向上拉，找到版本号即可