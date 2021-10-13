# 进阶：Android的launch_mode

activity有4个LaunchMode：

1. standard：标准
2. singleTop：在栈顶唯一
3. singleTask：在栈中唯一（也是**全局唯一**）
4. singleInstance：单独创建一个栈存储，**唯一性且独占性**，即独自霸占一个完整的task 

## 1. 最近任务栈

每个app都是一个任务，每个任务都有一个返回栈。app会将activity按照点击顺序保存在栈中，点击back键，里面的activity移次出栈并在栈顶时会在屏幕显示，如果返回栈中activity全部出去了，则该任务不复存在了，返回栈实际上就被删除了——但是在最近任务列表中还是**存在一个残影**。如果用户点击则又会**创建新的返回栈**，然后重新启动该app。

——可以看到此时的切换动画和普通的任务切换不一样

——所以**最近任务列表中的task不一定是存活的，可能是残影**

## 2. activity的跨进程、跨应用

即：在APP A中打开APP B的一个activity，就会将该activity_B直接加入到A中的返回栈栈顶，那么appB并不一定会被启动

——主要作用：实现activity的复用

规则：不同的app打开同一个activity，则会创建多个activity实例，然后放入对应的app的返回栈中

### 变体：

如果需要：appA打开appB中的activity，连带着需要启动appB，那么在最近任务列表中需要能够看到appA边上appB任务，且点击返回键，返回的是appB的之前activity

实现思路：将appB中的对应activity的launchMode设置为singleTask，那么进入该activity的时候，会将appB的返回栈全部堆到appA上面，然后该activity作为栈顶。

如果点击返回键，那么返回的是appB之前的activity，不断退出，直到appB的返回栈为空，则开始显示appA的activity

——可以看到：appA打开appB中的activity的切换动画变成：任务切换动画了，而不是activity的切换动画；appB的返回栈为空，则开始显示appA的activity，这边也是任务切换动画

——体现了，**不同任务可以叠成栈**，要求：在**前台task才可能进行叠加**，**如果是进入后台，则立即会被拆分，再回来的时候的还是拆分状态**

## 3. task前台到后台的切换

只有2种才会使得任务变成后台任务：

1. home键回到桌面
2. 查看最近任务列表，按方块键：在最近任务列表显示出来的时候已经到后台了，而不是等到应用切换后

## 4. activity在task之间跳转

**`allowTaskReparenting`**：设置为true，默认为false

那么activity可以在多个task之间跳归属

场景：在appA中打开appB的activity1，那么activity1在栈顶了；此时打开appB，那么activity1会从appA中跳转到appB，显示在appB的栈顶，而appA中已经不存在activity1了

优势：对于用户来说，由于没有发生task切换，所以它没有task切换的动画，而是丝滑的activity切换动画；且因为可能并没有启动新的B的task，也不会出现到后台拆分的问题，所以从后台切换回来后逻辑仍旧是正常的

缺陷：Android9、10存在bug，使能无效

——总结，由于上面2的使用和本身singleTask的特性，导致**一个task只有一个activity**，且保证了**只有一个task有该activity**，那么**singleTask修饰的activity全局唯一性**



singleInstance修饰的activity，在启动的时候，会创建activity，和单独的task，并且和singleTask类似，出现task堆叠——入场动画是task切换动画，点击返回直接去掉该task，所以出场动画是task切换动画

那么也就会存在如果堆叠期间切到后台，则堆叠会消失

如果singleInstance的activity中启动了另外app的activity，那么该activity所在的栈会移过来堆到这上面

==如果切到了后台（12:46）==，singleInstance不会出现在最近任务列表，在后台隐身存活，直到需要的时候调用`onNewIntent()`（而singleTask仍然在栈顶）

——**在最近任务列表中存在的task不一定存在，不存在的task也不一定是死了的**

## 5. taskAffinity

**一个task最多只有一个activity可以显示在最近任务列表中，是通过taskAffinity进行区分的，且不同的task的taskAffinity是不同的**

每个activity都有一个taskAffinity，取自application的taskAffinity，其是默认取自app的包名package

每个task都有taskAffinity，取自栈底的activity的taskAffinity

——可以定制taskAffinity，但是一般都是选择默认的，所以默认的就是taskAffinity就是包名

taskAffinity的作用：

1. app启动时，创建第一个activity，它带的taskAffinity（默认为包名）就赋值给该app对应的新的task的taskAffinity

2. 之后在该app中创建的activity都默认入栈

3. 如果activity的`launchMode="singleTask"`，那么需要去比较task的taskAffinity和该activity的taskAffinity是否一样

   - 如果一样，就直接入栈
   - 如果不一样，就去寻找和taskAffinity的task入栈，
   - 如果找不到，就创建一个新的task入栈（该task的taskAffinity就自然是该activit的taskAffinity）

   ​    ——然后将该task栈堆在该app的栈上

规则：多个task有相同的taskAffinity，那么最近任务列表只会展示最新展示过的那一个——所以singleInstance在最近任务列表中不存在是因为taskAffinity的问题 

参考：

1. https://www.bilibili.com/video/BV1CA41177Se

