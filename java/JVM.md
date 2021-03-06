# Android虚拟机 & Java虚拟机

- Android
	- Dalvik，针对Android优化的Java虚拟机，支持dex格式
		- 适合内存和CPU速度有限的系统
		- 同时运行多个虚拟机实例 
	- ART，“优化版”的Dalvik
		- 运行时更快，安装时预编译字节码（而Dalvik是每次都要编译+运行）
		- 缺点：安装变慢、占用空间变大（空间换时间） 
- JVM

# 对象头

- Mark Word
	- Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等等，占用内存大小与虚拟机位长一致 
- 类型指针
	- 虚拟机通过这个指针确定该对象是哪个类的实例 

# JVM内存模型

总共分为五个部分

1. 程序计数器 
2. 栈 
3. 本地方法栈 
4. 堆
5. 方法区

## 程序计数器

- 定义：记录当前线程正在执行的那一条字节码指令的地址
- 作用
	- 实现代码的流程控制
	- 多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行的位置 
- 特点
	- 是一块较小的存储空间
	- 线程私有。每条线程都有一个程序计数器，生命周期随着线程的创建而创建，随着线程的结束而死亡 
	- 是唯一一个不会出现OutOfMemoryError的内存区域

## 栈

- 定义：栈会为每一个即将运行的Java方法创建一块叫做“栈帧”的区域，每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息等内容
- 特点
	- 局部变量表的大小是不会发生改变的
	- 两种异常
		- StackOverFlowError，请求栈的深度超过当前Java虚拟机栈的最大深度的时候
		- OutOfMemoryError，申请栈时发现内存用光了
	- 线程私有

## 本地方法栈

本地方法栈和Java虚拟机栈实现的功能类似，只不过本地方法区是本地方法（native方法）运行的内存模型

## 堆

- 定义：堆是用来存放对象的内存空间
- 特点
	- 线程共享，整个Java虚拟机只有一个堆，所有的线程都访问同一个堆 
	- GC的主要场所
	- 细分
		- 新生代
			- Eden
			- From Survior
			- To Survior 
		- 老年代
	- 异常，OutOfMemoryError

> Q：为什么需要两个Survior区？
> 
> A：复制算法的代价就是要将内存折半，为了不浪费过多的内存，就划分了两块相同大小的内存区域survivor from和survivor to。在每次gc后就会把存活对象给复制到另一个survivor上，然后清空Eden和刚使用过的survivor	

## 方法区

- 定义：堆的一个逻辑部分，存放已经被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等，其中常量存储在运行时常量池
- 特点
	- 线程共享
	- GC效率低

[https://blog.csdn.net/u010425776/article/details/51170118](https://blog.csdn.net/u010425776/article/details/51170118)

# GC

## GC算法

### 标记-清除

1. 标记：标记的过程其实就是，遍历所有的GC Roots，然后将所有GC Roots可达的对象标记为存活的对象
2. 清除：清除的过程将遍历堆中所有的对象，将没有标记的对象全部清除掉

### 复制（适合新生代GC）

将原有的内存空间分为两块，每次只使用其中一块，在垃圾回收时，将正在使用的内存中的存活对象复制到未使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收

### 标记-整理（适合老年代GC）

1. 标记：它的第一个阶段与标记/清除算法是一模一样的，均是遍历GC Roots，然后将存活的对象标记
2. 整理：移动所有存活的对象，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收。因此，第二阶段才称为整理阶段

### 分代收集

根据对象的存活周期的不同将内存划分为几块儿。一般是把Java堆分为新生代和老年代：短命对象归为新生代，长命对象归为老年代。

- 少量对象存活，适合复制算法：在新生代中，每次GC时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成GC。
- 大量对象存活，适合用标记-清理/标记-整理：在老年代中，因为对象存活率高、没有额外空间对他进行分配担保，就必须使用“标记-清理”/“标记-整理”算法进行GC

## GC类型

logcat中可以看到GC的log信息

### `GC_FOR_ALLOC`

当堆内存不够的时候容易被触发，尤其是new一个对象的时候，很容易被触发到，所以如果要加速启动，可以提高dalvik.vm.heapstartsize的值，这样在启动过程中可以减少GC_FOR_ALLOC的次数。注意这个触发是以同步的方式进行的。如果GC后仍然没有空间，则堆进行扩张

### `GC_EXPLICIT`

这个gc是被可以调用的，比如system.gc, 一般gc线程的优先级比较低，所以这个垃圾回收的过程不一定会马上触发， 千万不要认为调用了system.gc，内存的情况就能有所好转

### `GC_CONCURRENT`
当分配的对象大小超过384K时触发，注意这是以异步的方式进行回收的.如果发现大量反复的Concurrent GC出现，说明系统中可能一直有大于384K的对象被分配，而这些往往是一些临时对象，被反复触发了。给到我们的暗示是：对象的复用不够。

### `GC_EXTERNAL_ALLOC` （在3.0系统之后被废了）
Native层的内存分配失败了，这类GC就会被触发。如果GPU的纹理、bitmap、或者java.nio.ByteBuffers的使用没有释放，这种类型的GC往往会被频繁触发。

## GC 收集器

> 基于JDK1.7 Update 14 之后的HotSpot虚拟机

### 新生代收集器

- Serial收集器（串行收集器）
	- 采用复制算法的新生代收集器
	- 单线程收集器
	- Stop The World
- ParNew收集器
	- Serial收集器的多线程版本
- Parallel Scavenge收集器
	- 并行的多线程新生代收集器
	- 可以通过配置参数，控制吞吐量   	  

### 老年代收集器

- Serial Old收集器
	- 与Serial收集器类似
	- 采用标记-整理算法的老年代收集器
- Parallel Old收集器
	- Parallel Scavenge收集器的老年代版本
- CMS收集器
	- 以获取最短回收停顿时间为目标的收集器
	- 基于“标记-清除”算法
	- 工作流程
		- 初始标记，仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”
		- 并发标记，进行GC Roots Tracing的过程，在整个过程中耗时最长。
		- 重新标记，为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要“Stop The World”。
		- 并发清除

### G1收集器

- 并行与并发
- 分代收集
- 空间整合
- 可预测的停顿

## 内存分配策略

- 对象优先在Eden区分配
- 大对象直接进入老年代
- 长期存活的对象将进入老年代
- 动态对象年龄判定，Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代
- 空间分配担保

> 什么是空间分配担保？
> 
> 1. 新生代采用的是复制收集算法，S0和S1始终只是用其中一块内存区，当出现GC后大部分对象仍然存活的话，就需要老年代进行分配担保，把survior区无法容纳的对象直接晋升到老年代
> 2. 那么这种空间分配担保的前提是老年代还有容纳的空间，一共有多少对象会活下来，在实际完成内存回收之前是无法明确知道的，所以只好取之前每次回收晋升到老年代对象容量的平均值大小作为经验值，与老年代的剩余空间比较，决定是否进行GC来让老年代腾出更多空间

## GC 触发条件

- Minor GC
	- Eden区空间满时 
- Full GC
	- System.gc()，建议执行GC
	- 老年代空间不足
	- 空间分配担保失败 

## STOP-THE-WORLD

为了保证GC过程不被其他工作线程修改

具体是标记阶段STW还是清除阶段STW，需要看具体的垃圾收集器回收策略，例如

- 串行收集器，整个过程都需要STW
- CMS收集器，初始标记、重新标记需要STW，并发标记和并发清除不需要

# 参考资料

- [http://www.cnblogs.com/smyhvae/p/4744233.html](http://www.cnblogs.com/smyhvae/p/4744233.html)
- [https://crowhawk.github.io/2017/08/15/jvm_3/](https://crowhawk.github.io/2017/08/15/jvm_3/)
- [https://mp.weixin.qq.com/s/Ts9esqNB6bf4SyseiXPsQw](https://mp.weixin.qq.com/s/Ts9esqNB6bf4SyseiXPsQw)