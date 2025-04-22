# Git分支管理

git通过分支来隔离不同的环境，比如生产环境、开发环境、测试环境等。开发时可以创建开发分支，测试通过后再与生产分支合并，即可实现不影响生产工作的同时开发新功能的效果

## 创建与合并分支

`master`是默认分支，在初始化git仓库时会由git自动创建

如果想创建一个开发环境的分支，可以通过

```
git switch -c dev
```

来创建，并直接切换到`dev`分支。`-c`的作用是创建，如果要创建一个分支，那么就要加上`-c`，如果仅仅用于切换分支，则不用加。`dev`是分支的名称，可以换为其他名称

这时候通过`git branch`查看所有分支及当前分支

```
$ git branch
* dev
  master
```

此时一共有2个分支，一个`master`，一个`dev`。`dev`前边有`*`说明当前所在分支是`dev`

假如master分支下，有一个`1.txt`文件，内容为

```
master
```

则当创建新的分支`dev`后，两个分支的内容此时是相同的。在`dev`分支下，如果我们将`1.txt`的内容修改为

```
master
dev
```

`add`并`commit`后，切换回`master`分支，会发现`1.txt`的内容仍然为`master`，说明分支之间互不影响

然后我们切换回`master`分支，再用`git merge`指令来合并分支

```
git merge dev
```

合并成功发现`master`分支下的`1.txt`内容也和`dev`分支下的内容一样了，合并完成后即可删除`dev`分支

```
$ git branch -d dev
Deleted branch dev (was 0f26606).
```

## 解决冲突

与上面的情况类似，假如master分支下，有一个`1.txt`文件，内容为`master`，当创建新的分支`dev`后，在`dev`分支下，我们将`1.txt`的内容修改为`dev`，`add`并`commit`后，切换回`master`分支，然后我们通过`git merge`指令来合并分支

```
$ git merge dev
Auto-merging 1.txt
CONFLICT (content): Merge conflict in 1.txt
Automatic merge failed; fix conflicts and then commit the result.
```

git会提示我们发生了冲突，这是因为我们刚才是把`master`改为了`dev`，git不知道应该保留哪个，这样自然会有冲突，所以需要自行修改冲突部分再`add`+`commit`

## 紧急修复Bug

这样一个场景：我们在`dev`分支上开发新的功能，这时生产环境中运行的`master`分支中发现了一个bug需要紧急修复。这时候我们需要暂缓新功能的研发，赶紧去修复bug。我们肯定要在一个非`master`分支上修复，然后在`master`分支上再合并。问题在于，这个非`master`分支要如何选择？

如果选择当前的`dev`分支，也许能够很快的修复bug，但是新功能还没有开发完，是不能合并到`master`上的，所以不成立；

那么可以选择在`master`分支下新建一个`bug`分支，然后在`bug`分支上修复完后，回到`master`合并`bug`，再回到`dev`继续开发新功能。这个听起来可以，所以先照做

假设`master`中的`1.txt`内容为

```
There is a bug
```

然后我们在`dev`分支上开发新功能，把内容改为了

```
There is a bug
deving...
```

没有`add`更没有`commit`，然后切换回`master`，居然发现`master`里的`1.txt`内容也和`dev`分支中的相同。怀疑是打开方式不对，我们回到`dev`分支，再进行一次`add`，再切回`master`，发现居然还是与`dev`分支相同的内容。再回到`dev`分支，这次`commit`，再切回`master`，发现终于回到修改`dev`分支之前的版本了

这是因为，git中**工作区与暂存区是共享的**，如果有内容留在工作区和暂存区中没有经过`commit`提交到`dev`分支的版本库，那么其他分支也可以看到`dev`分支在工作区和暂存区的修改

也许我们可以先在`dev`分支进行一次`commit`再切换回`master`，但是这样不好，因为未完成的工作不适合提交上去，这时候可以用另一个指令`stash`，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作

我们先在`dev`分支下执行`git stash`命令

```
$ git stash
Saved working directory and index state WIP on dev: ebda2f7 1
```

git提示我们工作区和暂存区都被存储在了一个栈结构中，这时`dev`分支下的内容**回到了上一次`commit`后的内容**，如果在创建`dev`分之后没有进行任何提交，就回到了与`master`相同的状态

这时候我们回到`master`，再创建`bug`分支，并在`bug`分支中修复bug后回到`master`合并，最后删除`bug`分支。执行的指令如下

```
MINGW64 /d/git测试/test1 (dev)
$ git stash
Saved working directory and index state WIP on dev: ebda2f7 1

MINGW64 /d/git测试/test1 (dev)
$ git switch master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

MINGW64 /d/git测试/test1 (master)
$ git switch -c bug
Switched to a new branch 'bug'

// 修复了bug

MINGW64 /d/git测试/test1 (bug)
$ git add *

MINGW64 /d/git测试/test1 (bug)
$ git commit -m 修复bug
[bug db53862] 修复bug
 1 file changed, 1 insertion(+), 1 deletion(-)

MINGW64 /d/git测试/test1 (bug)
$ git switch master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

MINGW64 /d/git测试/test1 (master)
$ git merge bug
Updating ebda2f7..db53862
Fast-forward
 1.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
 
MINGW64 /d/git测试/test1 (master)
$ git branch -d bug
Deleted branch bug (was db53862).
```

这时候`master`中的`1.txt`内容为

```
There is no bug
```

修复完bug就要回到`dev`分支继续开发了，回去之后，这时的内容是上次`git stash`之后的内容，我们可以通过`git stash list`查看栈中存储内容的列表，然后通过`git stash pop`恢复最上边的那个状态，并在list中清除该状态（一般也只会有一个stash）

```
MINGW64 /d/git测试/test1 (dev)
$ git stash list
stash@{0}: WIP on dev: ebda2f7 1

MINGW64 /d/git测试/test1 (dev)
$ git stash pop
On branch dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   1.txt

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (dd4f71efbea7c847a764e46ec84bf1c1c3bd1ddd)
```

原来开发中的功能是恢复了，但是之前修改的bug还存在`dev`分支中，如何解决？再手动解决一次？这样不好，git专门提供了一个`cherry-pick`命令，让我们能复制一个特定的提交到当前分支，使用方法如下

```
MINGW64 /d/git测试/test1 (dev)
$ git cherry-pick db53862
error: Your local changes to the following files would be overwritten by merge:
        1.txt
Please commit your changes or stash them before you merge.
Aborting
fatal: cherry-pick failed
```

`cherry-pick`后的内容是在`bug`分支上提交后的版本号。结果git报错了，是因为我们的执行顺序反了。实际上**应该先`cherry-pick`再`shash pop`**

回到刚修复完bug，从`master`回到`dev`的时刻，先

```
MINGW64 /d/git测试/test1 (dev)
$ git cherry-pick db53862
[dev d09e49f] 修复bug
 Date: Thu Nov 4 13:44:19 2021 +0800
 1 file changed, 1 insertion(+), 1 deletion(-)
```

这时候把之前修复的bug也在`dev`分支中修复了，然后再`git stash pop`

```
MINGW64 /d/git测试/test1 (dev)
$ git stash pop
Auto-merging 1.txt
CONFLICT (content): Merge conflict in 1.txt
The stash entry is kept in case you need it again.
```

结果又提示我们有冲突，需要手动解决，并且为了防止我们还需要那个stash，所以进行了保留，我们后面可以通过`git stash drop`来仅删除操作

我们在手动合并完之后就可以继续开发了

