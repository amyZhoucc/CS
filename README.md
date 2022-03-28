:wave:这个是zcc的面试准备仓库，里面东西还不全，正在加紧学习补充中:running:.....（2022.3.28更新！）

这个仓库主要是基于Java:coffee:的，可能还会有一点C相关的知识？（C语言是我初恋:candy: ）

目前有的：

|                            Java                            |                             OS                             |                             计网                             |                             JVM                              |                           Leetcode                           |                          计算机基础                          |                           Android                            |
| :--------------------------------------------------------: | :--------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [:coffee:](https://github.com/amyZhoucc/CS/tree/main/java) | [:computer:](https://github.com/amyZhoucc/CS/tree/main/os) | [:cloud:](https://github.com/amyZhoucc/CS/tree/main/%E8%AE%A1%E7%BD%91) | [:japanese_ogre:](​https://github.com/amyZhoucc/CS/tree/main/jvm) | [:lion:](​https://github.com/amyZhoucc/CS/tree/main/leetcode_%E5%A4%A7%E6%88%98300%E9%A2%98) | [:earth_asia:](​https://github.com/amyZhoucc/CS/tree/main/%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86) | [:robot:](https://github.com/amyZhoucc/CS/tree/main/android%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0) |
| **设计模式** | **Kotlin** | **前端基础** | **数据库** | **工具** | **面筋** |  |
| [:deer:](https://github.com/amyZhoucc/CS/tree/main/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F) | :man: |                         :spider_web:                         | :memo: | [:slot_machine:](https://github.com/amyZhoucc/CS/tree/main/%E5%B7%A5%E5%85%B7) | [:trumpet:](https://github.com/amyZhoucc/CS/tree/main/%E7%83%A4%E9%9D%A2%E7%AD%8B) |  |
由于是零基础，所以都是从零开始，整理的不好且速度比较慢，而且我也不知道哪边缺了，只能去其他朋友的网站偷学，如果有热心市民老王，麻烦抓一下虫:

网上很火的，cyc、JavaGuide我基本是过了一遍理论之后作为查漏补缺用的，个人不喜欢背八股文所以大部分都只是做了理解，所以内容会很繁琐，但是理解了就不用背很多东西还是很爽的

目前的进度

1. Java基础：

   :bookmark_tabs:进度：参考的书籍算是整理过一遍了，总体也复习了一遍，抓了一些bug，后面的任务就是**复习 + 补充**，主体框架和内容应该变动不大了。

   :book:：[《Java编程的逻辑》](https://weread.qq.com/web/reader/b51320f05e159eb51b29226)我觉得写的挺好的，从Java基础到容器、并发、流等很全，也会讲到底层的一些知识，我啃了好久好久好久（大概一个世纪吧）但是感觉看这本书的人不多，如果有基础的可以很快看完，可以去看看；

   :book:：[《Java并发编程的艺术》](https://weread.qq.com/web/reader/247324e05a66a124750d9e9)网上查并发相关的blog大部分都能发现参考了这本书，但是这本书有且只有这一版，里面有些内容过时了，eg：ConcurrentHashMap它是按照Java7解释的，Java8已经有了很大的改进，但是了解并发这一块还是很不错的。

   :book:：[美团技术团队](https://tech.meituan.com/)，在并发这一块一些文章写的很好，可以仔细看看（你永远可以相信它）

   其他的内容，都在每份文档里面有备注参考的博客

   :flags: 后面打算看[《Java并发实现原理》](https://weread.qq.com/web/reader/6de3271071dbddc06de1a75kc81322c012c81e728d9d180)，[《Java并发编程之美》](https://weread.qq.com/web/reader/81c32b507184869281c2a23kc81322c012c81e728d9d180)

   :flags: 补充一些关于集合类等的源码知识（去抄作业）

2. 操作系统：

   :bookmark_tabs:进度：大头书看并理解了一遍，但是并未实现二次复习和复盘

   :book: [《操作系统概念》](https://book.douban.com/subject/30297919/)，建议看英文原版，我是时间比较紧张，直接配合PPT + 中文版，中文版里面错误还有点多，但是大意不差。

   :flags: 后面打算看[Linux相关的书，《鸟哥的Linux私房菜》](http://linux.vbird.org/)，[指令](https://www.runoob.com/w3cnote/linux-common-command-2.html)等记一下

   :flags: cmake、ninja等编译过程学习一下（因为项目相关）

3. 计网：

   :bookmark_tabs:进度：复习次数比较多了，后面把pdf里面的补充和抓到的bug都誊到markdown上面

   :book: [《计算机网络-谢希仁》](https://weread.qq.com/web/reader/af532c005a007caf51371b1) 经典的基础入门书（第一次看的我也啃了好久好久）

   :book:[《小林coding-整理的关于HTTP、TCP、IP知识点》](https://blog.csdn.net/qq_34827674/category_9811520.html) 很好的罗列出了面试易考的点，且画了很多图，超级方便记忆（我有将它的内容截图整理放到我的文档中）

   :flags: 后面打算看一下[《Wireshark》](https://weread.qq.com/web/reader/86c329705b208086cbdf910)，算是一个实践类型的书

4. JVM：

   :bookmark_tabs:进度：看了Java的内存布局情况、垃圾收集器、并发

   :book:：[《深入理解Java虚拟机》](https://weread.qq.com/web/reader/9b832f305933f09b86bd2a9) 几乎是最权威的书了，但是难度太大，只能挑着看了

   :flags:调优啥的都不看了，直接看[part3](https://weread.qq.com/web/reader/9b832f305933f09b86bd2a9k45c322601945c48cce2e120)

5. leetcode

   :bookmark_tabs:进度：我还是个菜鸟中的菜鸟，目前是剑指75题已经认真过了2遍了

   :book:：[《剑指Offer》](https://book.douban.com/subject/6966465/)，里面的内容 + 题都已经过了一遍

   :flags: 高频前三百题！！！！

   :flags:[汇总](https://codetop.cc/#/home)有人将牛客面经上的题进行了整理，点击就能跳转到leetcode指定的题，都是面试手撕的算法题，需要过一遍

   :flags: 算法基础的数据结构、排序、查找等都需要整理并复现一下

   正如师兄说的，算法题不能落下，要天天做不然容易手生，这样做的就更烂了，想师兄学习！:respect!

6. 计算机基础

   这个是一个很难以准备的东西，全看自己的计算机素养和~~编（不是）~~的能力了

   只能是遇到啥整理啥

   eg：回调函数、编译过程啥的

7. **Android**

   :bookmark_tabs:进度：Android第一行代码看完2遍左右了，但是只能说能简单的用，对于自定义view和

   :book:：[《Android第一行代码》](https://weread.qq.com/web/reader/7c532360718ff6317c5255d) Java版本和kotlin版本已经都完整过了一遍，并且重点部分又过了一遍，里面的实践已开发完成

   :book:：[《Android开发艺术探索》](https://weread.qq.com/web/reader/9d932320716a2b159d9b881)部分章节已经看过，但是没有做好笔记，抓紧时间看一遍

   :book:《郭霖的公众号》会收集很好的文章（毕竟大佬筛选过，看起来放心），争取好的文章都过一遍（目前还是在技术研究期，不会涉及到新技术研啥的，用到啥学啥）

   :flags:

8. 设计模式

   :bookmark_tabs:进度：head first已经过了一遍，但是感觉掌握的不咋地，买了[极客时间的课](https://time.geekbang.org/column/intro/100039001)，看起来不错

   :book:：《head first design patterns》确实是一本很有趣的书，能让枯燥抽象的设计模式讲的详细，如果想靠它成为专家是不可能的，只能做了解

   :flags: 将23种设计模式都了解一遍，对其中几种重要的都需要了解它的实例

9. **kotlin**

   因为业务是kt的，所以需要学习，但是没有找到合适的书籍，买了[课]()

10. **前端**

   前端客户端不分家嘛，还是都要学的，但是有点杂，需要收集学习的重点

   :bookmark_tabs:进度：TypeScript（JavaScript的基础上）

   :flags: 

9. 数据库

   :bookmark_tabs:进度：没学过（苦涩:sweat_smile: ），毫无了解，用过一点select等的

   :flags: 根据面经，去了解一点，够做题和面试就行了

10. 工具

    主要是包括：git、markdown的使用（简历里面写到了），需要了解并能够说出来

    :bookmark_tabs:进度：Git的基本指令已经整理过了，还需要再复习，再加点进阶的东西；markdown已经整理一遍有哪些内容了

11. 面经

    :bookmark_tabs:进度：从777那边抄作业抄了不少，后面需要加上自己实际面试的内容，和牛客上无穷无尽的面经

——zcc要毕业啦，继续学习，不断进步，有啥学习紧张都会更新哒，冲冲冲！:rocket:
