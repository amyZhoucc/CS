# Git

Git是分布式版本控制系统

集中式版本控制系统最大的毛病就是必须联网才能工作，eg：SVN

分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

修改的其实只能跟踪文本文件的改动，而图片、视频这些二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化。eg：Microsoft的Word格式是二进制格式，因此，版本控制系统是没法跟踪Word文件的改动的

## 创建库

1. `git init`
2. `git add <file>`
3. `git commit -m <message>`

## 看状态

1. `git status`：看仓库当前状态
2. `git diff <文件名>`：比较文件修改之前和现在的差别

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210519143721560.png" alt="image-20210519143721560" style="zoom: 50%;" />

（上面的文件是已经被添加过了，但是发生了修改而且还未提交，而下面的文件是从来没有被添加过，所以它的状态是Untracked。）

## 版本回退

`git log`查看命令。

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20210519144144593.png" alt="image-20210519144144593" style="zoom:50%;" />

`commit id`版本号，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示

`git reset --hard HEAD^/ git reset --hard commitId`回退到上一个版本。版本号没必要写全，前几位就可以了，Git会自动去找。

## 工作区和暂存区

就是整个文件夹，那么就是工作区

版本库：`.git`就是git的版本区，最重要的就是stage（或者叫index）的暂存区。Git为我们自动创建的第一个分支`main`，以及指向`main`的一个指针叫`HEAD`。

1. `git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

2. `git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`main`分支上提交更改。

`git add`命令实际上就是把要提交的所有修改放到暂存区（Index），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。

每次修改，如果不用`git add`到暂存区，那就不会加入到`commit`中。

## 撤销修改

`git checkout -- file`可以丢弃工作区的修改

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

1. `readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

   `git checkout -- readme.txt`表示把`readme.txt`文件在**工作区的**修改全部撤销——和版本库中的内容一致

2. `readme.txt`已经添加到**暂存区**后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态；

   先使用`git restore --staged Git.md`将暂存区的修改撤销掉，重新放回工作区

   然后再使用`git checkout -- readme.txt`就能回到未添加到暂存区之前
   
   这个文件回到最近一次`git commit`或`git add`时的状态。用`git status`查看一下，修改只是添加到了暂存区

——总结：如果该文件未添加到暂存区，那么直接使用checkout--，如果已经添加到暂存区，那么

## 删除

对于普通的文件，`rm <file>`，如果该文件没有被同步到版本库中，那么git并没有感受到。

而如果同步到版本库，那么rm删除就会被感应到，那么`git status`就会发现删除。

如果要从版本库中删除该文件，那么`git rm -f <file>`，然后再次`git commit -m "ddf"`更新即可。

那么工作区和版本区的内容都一样了。

先手动删除文件，然后使用`git rm <file>`和`git add <file>`效果是一样的。

可以用前面的`git checkout -- file`可以恢复。

**从来没有被添加到版本库就被删除的文件，是无法恢复的**

## 远程库

remote表示远程库。添加后，远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是`origin`这个名字一看就知道是远程库。

下一步，就可以把本地库的所有内容推送到远程库上：

要关联一个远程库，使用命令`git remote add origin <url>` remote表示远程库，远程库的名字默认是origin

命令`git push -u origin main`第一次推送master分支的所有内容；

删除一个远程库：`git remote rm <name>`

看远程库上的版本：`git remote -v`

## 创建和合并分支

`git checkout -b dev`，表示创建并切换

`git branch`可以用来查询当前分支。

`git checkout master`切换分支

`git merge xxx`命令用于合并指定分支到当前分支，eg：当前分支是main，那么就是将xxx分支合并到main分支上。

`git branch -d dev`删除分支

也可以将checkout用switch进行替换：

1. `git switch -c dev`：创建分支dev，并且将其切换到dev上
2. `git switch`：就是普通的切换分支

## 冲突分支

一个分支对一个文件进行修改，另一个分支也对该文件进行修改，如果要合并两个分支，及`git merge xxx`，那么会存在冲突。

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

```
<<<<<<<到=======是在当前分支合并之前的文件内容
=======到>>>>>>> 是在其它分支下修改的内容
需要在这个两个版本中选择一个，然后把标记符号也要一起删除
```

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

用`git log --graph`命令可以看到分支合并图。