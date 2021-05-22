# Git指令记录

## 新建仓库

分远程仓库的创建、本地仓库的创建，并且上传

1. 手动在远程仓库创建一个新的库，并自动生成README.md文件

2. 本地创建

   - 如果该库是空的，就需要新建一个文件夹，并`cd`到该目录下；否则，就直接`cd`到该目录

   - `git init`——初始化

   - `git add .`

   - `git commit -m "explain"`

   - `git remote add origin `+远程仓库地址

   - `git pull --rebase origin master`——本指令执行完，本地就会出现一个README.md

     由于本地文件常常没有readme，所以需要先将本地和远程仓库的内容一致，所以先`pull`，再`push`。

   - `git push -u origin master`——再将代码上传到远程





## Git指令记录

1. git提交：`git commit`

   `-m "explain"`：可选的，表示提交的时候添加备注

   git仓库的**提交记录都是保存目录下的所有文件快照**（snapshot），类似于将整个文件目录复制粘贴

   git希望每次提交都是轻量的，所以每次提交，不是全部复制后替换，而是**将当前版本与之前的版本进行比较**，将**差异打包**作为实际的提交记录（如果是初次提交就没有办法进行比较）。

   git**保存提交记录**，方便回溯。

   图示：c2是最新的提交记录，c2的父节点是c1，父节点是当前提交中变更的基础：

   <img src="pic\image-20200627171457165.png" alt="image-20200627171457165" style="zoom: 67%;" />

2. git分支：`git branch <branch_name>`

   如果没有加后面的`branch_name`，就会显示该库中的各个分支（查询指令）

   用来创建分支（就是某次提交记录）——操作的对象是当前分支，创建分支不会造成存储或内存上的开销，并且不同分支方便维护

   **创建分支意味着，想基于该提交记录和其之前的提交记录来提交新的内容**

   图示：表明c1（当前提交），被命名为*bugFix*分支

   <img src="pic\image-20200627172804350.png" alt="image-20200627172804350" style="zoom: 67%;" />

3. git切换分支：`git checkout <branch_name>`

   切换到指定的分支上，默认是master分支

   `git checkout -b <branch_name>`：新建分支并且切换到该分支上	

4. git分支合并：`git merge <branch_name>`、`git rebase`

   分支合并一般用在，新建一个分支，在该分支上开发完毕后，并回到主线（这样就不会影响主线的主要功能）

   `git merge <branch_name>`：会产生一个新的提交记录，该节点有两个父节点，含义就是将这两个父节点（父提交记录）及其所有祖先节点都包括进来。是指定分支（`branch_name`）合并到当前分支

   eg：`git merge bugFix`：将分支c2、c3都包含了，产生了节点c4，并且属于master。该master包含了所有的提交记录。（但是bugFix没有被记录）

   <img src="pic\image-20200627213149189.png" alt="image-20200627213149189" style="zoom:50%;" />

   `git checkout bugFix; git merge master`：切换分支bugFix，然后将master合并到bugFix

   （那么所有节点都包含了所有修改）

   <img src="pic\image-20200627214246401.png" alt="image-20200627214246401" style="zoom:50%;" />

   `git rebase <branch_name>`：将当前分支合并到指定分支上。具体实现是，取出一系列的提交记录，复制然后在指定地方逐个粘贴。与merge的区别就是，线性的创造提交历史

   eg：

   初始：有两个分支：master、bugFix（为当前分支）

   <img src="pic\image-20200627220552785.png" alt="image-20200627220552785" style="zoom:50%;" />

   之后：git rebase master：将当前分支（bugFix）合并到master上，所以就造成了在master更新的假象。（c3依然存在)，c3' 是c3的副本，但是此时，master没有更新

   <img src="pic\image-20200627220513644.png" alt="image-20200627220513644" style="zoom:50%;" />

   之后只需要切换分支到master，然后master合并到bugFxi，然后就更新了所有节点

   `git checkout master; git rebase bugFix`：

   <img src="pic\image-20200627221742664.png" alt="image-20200627221742664" style="zoom: 50%;" />

5. 本地创建远程仓库的拷贝：`git clone <url>`

   将指定的远程仓库的文件下载，并且结构和远程仓库的一致。并且本地会产生一个.git文件夹（存放本地提交记录等？？？）

   本地仓库的分支——**远程分支**

   远程分支，反映了远程仓库的状态（是指上一次和远程仓库通信时的状态）

   eg：左侧是本地分支情况，右侧是远程分支

   <img src="pic\image-20200627231935222.png" alt="image-20200627231935222" style="zoom: 67%;" />

   可以发现，本地多了一个`o/master`（o = origin）分支（这个就是远程分支）

   远程分支，会在检出时自动进入分离HEAD的状态，那么更新远程分支后再用远程分享成果

   `o/master`，只有在远程仓库中的对应分支更新了才会更新

   eg：默认是master分支，然后执行`git commit`：然后增加了一个提交节点c3

   <img src="pic\image-20200627233354537.png" alt="image-20200627233354537" style="zoom:50%;" />

   然后执行`git checkout o/master`：切换当前分支

   <img src="pic\image-20200627233604691.png" alt="image-20200627233604691" style="zoom:50%;" />

   再执行一次commit：就会检出分离HEAD状态

   <img src="pic\image-20200627233738730.png" alt="image-20200627233738730" style="zoom: 50%;" />

6. 从远程仓库获得数据：`git fetch`

   `git fetch`：完成了 从远程仓库下载本地仓库缺失的提交记录 + 更新远程分支指针（o/master） => 就是更新到最新状态（同步了）

   但是，它不会更新本地仓库的状态，即**不会更新master分支**（只是下载，而没有更新）

   图示：执行前，远程有2个提交记录，本地没有更新

   <img src="pic\image-20200627233954749.png" alt="image-20200627233954749" style="zoom:50%;" />

   执行后：`git fetch`：c2、c3都被更新到本地，并且`o/master`也被更新

   <img src="pic\image-20200627234121464.png" alt="image-20200627234121464" style="zoom:67%;" />

7. 将变化更新到本地仓库：`git pull`

   初始状态：远程仓库有最新的提交记录，没有更新到本地

   <img src="pic\image-20200627235938331.png" alt="image-20200627235938331" style="zoom:67%;" />

   执行之后：`git fetch; git merge o/master`：`git fetch`将远程分支更新到本地c3，并且更新`o/master`；

   ​			`git merge o/master`将`o/master`合并到当前分支master，变成了c4

   <img src="pic\image-20200628000218445.png" alt="image-20200628000218445" style="zoom: 67%;" />

   **git pull 就能一次实现上述两个操作**：fetch+merge

8. 上传本地变更到远程仓库：`git push`

   `git push`，会将本地仓库的更新到对应的远程仓库，然后在远程仓库合并新的提交记录（这样其他人也能看到新成果）

   初始状态：本地仓库有变化（已经commit）

   <img src="pic/image-20200628001137934.png" alt="image-20200628001137934" style="zoom:67%;" />

   执行之后：将本地提交分支c2上传到远程仓库，并且更新远程的master，更新本地的o/master => 所有分支都同步了

   <img src="pic\image-20200628001245977.png" alt="image-20200628001245977" style="zoom:67%;" />

   如果本地有更新，远程仓库也有更新，那么需要先pull再push

   `git pull; git push`

9. 

