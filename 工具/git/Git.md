# Git

Git是分布式版本控制系统

集中式版本控制系统最大的毛病就是必须联网才能工作，eg：SVN

分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

修改的其实只能跟踪文本文件的改动，而图片、视频这些二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化。eg：Microsoft的Word格式是二进制格式，因此，版本控制系统是没法跟踪Word文件的改动的

## 创建库

1. ==**`git init`**==
2. ==**`git add <file>`**==
3. ==**`git commit -m <message>`**==

## 看状态

1. ==**`git status`**==：看仓库当前状态，哪些文件被修改了、哪些文件被新建了，都还没有
2. ==**`git diff <文件名>`**==：比较文件修改之前和现在的差别

<img src="pic\image-20210519143721560.png" alt="image-20210519143721560" style="zoom: 50%;" />

（上面的文件是已经被添加过了，但是发生了修改而且还未提交，而下面的文件是从来没有被添加过，所以它的状态是Untracked。）

## 版本回退

==**`git log`**==查看命令。

<img src="pic\image-20210519144144593.png" alt="image-20210519144144593" style="zoom:50%;" />

`commit id`版本号，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示

==**`git log --oneline`**==：能够看到log的简略信息，commitID也只有简单的前7位。

==**`git reset --hard HEAD^/ git reset --hard commitId`**==回退到上一个版本。版本号没必要写全，前几位就可以了，Git会自动去找。

## 工作区和暂存区

就是整个文件夹，那么就是工作区

版本库：`.git`就是git的版本区，最重要的就是stage（或者叫index）的暂存区。Git为我们自动创建的第一个分支`main`，以及指向`main`的一个指针叫`HEAD`。

1. `git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

2. `git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`main`分支上提交更改。

`git add`命令实际上就是把要提交的所有修改放到暂存区（Index），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。

每次修改，如果不用`git add`到暂存区，那就不会加入到`commit`中。

## 撤销修改

==**`git checkout -- file`**==可以丢弃工作区的修改

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

如果要从版本库中删除该文件，那么**`git rm -f <file>`**，然后再次`git commit -m "ddf"`更新即可。

那么工作区和版本区的内容都一样了。

**先手动删除文件，然后使用`git rm <file>`和`git add <file>`效果是一样的。**

可以用前面的`git checkout -- file`可以恢复。

**从来没有被添加到版本库就被删除的文件，是无法恢复的**

## 远程库

remote表示远程库。添加后，远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是`origin`这个名字一看就知道是远程库。

下一步，就可以把本地库的所有内容推送到远程库上：

要关联一个远程库，使用命令==**`git remote add origin <url>`**==，remote表示远程库，远程库的名字默认是origin

命令==**`git push -u origin main`**==第一次推送master分支的所有内容；

删除一个远程库：==**`git remote rm <name>`**==

看远程库上的版本：`git remote -v`

## 创建和合并分支

==**`git checkout -b dev`**==，表示创建并切换

（==**`git checkout -b dev origin/dev`**== 从远程拉去分支到本地分支中）

`git branch`可以用来查询当前分支。

==**`git checkout master`**==切换分支

==**`git merge xxx`**==命令用于合并指定分支到当前分支，eg：当前分支是main，那么就是将xxx分支合并到main分支上。

==**`git branch -d dev`**==删除分支

如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。——大写的-D，表示强制删除

也可以将checkout用switch进行替换：

1. `git switch -c dev`：创建分支dev，并且将其切换到dev上
2. `git switch`：就是普通的切换分支

## 冲突分支

发生冲突：一个分支对一个文件进行修改并提交（add和commit），另一个分支也对这个文件的相同位置进行了修改并提交，那么在合并的时候，git不知道该如何选择。如果是不同的位置，那么完全可以进行合并选择的。

`git merge xxx`，那么会存在冲突。当git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

> `<<<<<<<`到`=======`是在当前分支合并之前的文件内容
> `=======`到`>>>>>>>` 是在其它分支下修改的内容
> 需要在这个两个版本中选择一个，然后把标记符号也要一起删除

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交`git add .`那么可以将内容直接push上去了

用`git log --graph`命令可以看到分支合并图。

## 分支管理

前面都是`fast forward`快速合并的模式，`--no-ff`方式的`git merge`——Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

## Stash

Git还提供了一个`stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作

1. 先将当前现场保存下来：**`git stash`**

2. 假定需要在`main`分支上修复，就从`main`创建临时分支：切换分支，然后在上面创建临时分支，然后进行代码修改。然后切回到分支，对临时分支进行合并，然后删除临时分支

   ```
   git switch main
   
   git switch -c issue-01
   git add .
   git commit -m "fix bugs on issue-01"
   
   git switch main
   git merge -no-ff -m "merge bugs branch" issue-01
   
   git switch dev
   ```

3. ==**`git stash list`**==可以看当时保护的现场

4. 恢复现场：一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；

   另一种方式是用`git stash pop`，恢复的同时把stash内容也删了

但是存在，如果dev分支也存在和main同样的bug，要在dev上修复，我们只需要把`4c805e2 fix bug 101`这个提交所做的修改“复制”到dev分支。注意：我们只想复制`4c805e2 fix bug 101`这个提交所做的修改，并不是把整个master分支merge过来。

为了方便操作，Git专门提供了一个`cherry-pick`命令，让我们能复制一个**特定的提交到当前分支**

`git cherry-pick 4c805e2`——给dev分支做了一次提交，主要就是将该commitId的分支合并到当前的dev分支上。

## 多人协作模式

查看远程库信息，使用`git remote -v`；本地新建的分支如果不推送到远程，对其他人就是不可见的

多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交add+commit；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

## rebase操作

背景：从一个分支中分出两根，然后它们各自又提交了更新，此时需要将这两个分支进行合并，然后回归到原分支，eg：main

<img src="pic\image-20210519225050243.png" alt="image-20210519225050243" style="zoom:67%;" />

`merge` 命令：它会把两个分支的最新快照（`C3` 和 `C4`）以及二者最近的共同祖先（`C2`）进行三方合并，合并的结果是生成一个新的快照（并提交）——C5

<img src="pic\image-20210519224745652.png" alt="image-20210519224745652" style="zoom:67%;" />

另一个操作是：

```
git switch experiment			切换到上面分支c4
git rebase master				将experiment的变化应用到master分支

git switch master				切换到主分支
git merge experiment			然后将experiment合并到主分支上——这个就是快速合并
```

——原理：先找到它们的共同祖先，然后对比当前分支相对于该祖先的变化，提取相应的修改形成临时文件，然后将当前分支指向C3，然后将临时文件的修改应用到C3中。于是内容变成了如下的原理：

<img src="pic\image-20210519225158469.png" alt="image-20210519225158469" style="zoom:50%;" />

——把分叉的提交历史“整理”成一条直线，看上去更直观。缺点是本地的分叉提交已经被修改过了。

```console
git rebase --onto master server client
取出 client 分支，找出它从 server 分支分歧之后的补丁， 然后把这些补丁在 master 分支上重放一遍，让 client 看起来像直接基于 master 修改一样——那么client就直接和master相连了

git checkout master
git merge client		//将client合并到master

git rebase master server		// 将server合并到master——就被续到了master的最后
```

## 打tag

发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的**历史版本取出来**。所以，标签也是版本库的一个快照。

Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针——用tag主要是方便人记忆和沟通

`git tag v1.0`就在当前分支的当前commit下打了tag——默认标签是打在最新提交的commit上的

对于历史的commit，找到历史提交的commit id，然后打上就可以了

`git tag v0.9 f52c633`

还可以创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字：

`git tag -a v0.1 -m "version 0.1 released" 1094adb`

注意：**如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。**

删除：`git tag -d v0.1`因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

push到远程：`git push origin v1.0`/`git push origin --tags`推送全部的尚未推送的tag到远程

如果标签已经推送到远程，要删除远程标签就麻烦一点

1. 先从本地删除：`git tag -d v1.0`
2. 在删除远程的：`git push origin :refs/tags/v0.9`

## 禁止一些文件夹、文件上传

一般在add文件的时候，都是`git add .`，就会默认将所有修改都放入暂存区

**创建一个`.gitignore`文件**，然后直接将需要屏蔽的文件夹（文件）写入，并且将本次更新commit，那么对应的文件夹（文件）就不会上传了，只在本地有效——这个主要用于本地环境的配置，而不需要把这些文件上传