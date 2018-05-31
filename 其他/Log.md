# 2018.5.31

性能优化

- 内存优化
	- 内存泄漏
		- 工具，MAT，LeakCanary，Logcat GC Log 
	- 内存溢出OOM
- UI优化
	- 布局优化，减少层级，过度绘制
	- 绘制优化，onDraw方法中避免对象的频繁创建
- 代码优化
	- 同步变异步，减少ANR
	- 使用高效的数据结构  
- 网络优化
	- 根据不同的网络状况，执行不同的策略，例如wifi充电状态下，下载开屏资源等等
	- 弱网优化，界面先反馈，再执行后续操作等等 
- 电量优化
	- 网络请求前，判断网络类型，wifi环境下的传输的电量消耗会小于移动网络下的传输 
- 启动优化
	- 冷启动
		- 第三方组件的延迟初始化
			- 子线程中初始化
			- 使用前再初始化
		- 减少初始网络请求   
	- 热启动
		- 优化MainActivity的布局绘制、初始化等等 

# 2018.5.30

硬件加速开启后，绘制的计算工作由 CPU 转交给了 GPU

硬件加速效率更高，主要有三个原因：

- 本来由 CPU 自己来做的事，分摊给了 GPU 一部分，自然可以提高效率
- 相对于 CPU 来说，GPU 自身的设计本来就对于很多常见类型内容的计算（例如简单的圆形、简单的方形）具有优势
- 由于绘制流程的不同，硬件加速在界面内容发生重绘的时候绘制流程可以得到优化，避免了一些重复操作，从而大幅提升绘制效率

# 2018.5.28

调用系统默认相机，如果不设置MediaStore.EXTRA_OUTPUT参数，则不会输出高质量图片

临时文件如何处理？

[https://blog.csdn.net/u010296640/article/details/72731324](https://blog.csdn.net/u010296640/article/details/72731324)

# 2018.5.17

- invalidate、postInvalidate、requestLayout应用场景
	- requestLayout
		- 子View调用requestLayout方法，会标记当前View及父容器，同时逐层向上提交，直到ViewRootImpl处理该事件（责任链模式），ViewRootImpl依次调用scheduleTraversals、performTraversals，从而调用三大流程，对于每一个含有标记位的view及其子View都会进行measure、layout、draw
	-  invalidate
		- 与requestLayout类似，区别在于，没有添加measure、layout的标志位，所以只会执行draw的流程
	-  postInvalidate
		- 与invalidate作用一样，但postInvalidate可以在非UI线程中执行

- 加载阶段读入.class文件，class文件是二进制么？为什么需要使用二进制的方式？
	- class文件是一种8位字节的二进制流文件， 各个数据项按顺序紧密的从前向后排列， 相邻的项之间没有间隙， 这样可以使得class文件非常紧凑， 体积轻巧， 可以被JVM快速的加载至内存， 并且占据较少的内存空间  

# 2018.5.14

- ViewStub实现原理
	- ViewStub是一个默认不显示，不绘制，宽高为0的View
	- 源码中的几个核心代码

	```
	public ViewStub(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
		super(context);
		...
		setVisibility(GONE);
		setWillNotDraw(true);
	}
	
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(0, 0);
    }
    
    // setMeasuredDimension 设置当前View的宽高
	``` 
	
- Relativelayout和LinearLayout在实现效果同等情况下选择使用哪个？为什么？
	- View绘制的三个阶段，onMeasure、onLayout、onDraw，需要从这三个阶段入手分析
	- 两种实现方式，onLayout、onDraw的执行时间基本一致 
	- RelativeLayout会对子View做两次measure。这是由于RelativeLayout是基于相对位置的，而且子View会在横向和纵向两个方向上分布，因此，需要在横向和纵向分别进行一次measure过程
	- LinearLayout只进行横向或者纵向的measure，因此measure的时间要比RelativeLayout短（不过，在设置了weight时，会对weight进行两次measure）

- view的工作原理及measure、layout、draw流程 
	- View 的绘制流程是从 ViewRoot 的 performTraversals 方法开始的，它经过 measure、layout、draw 三个过程才能最终将一个 View 绘制出来，其中 measure 用来测量 View 的宽和高，layout 用来确定 View 在父容器的放置位置，而 draw 则负责将 View 绘制在屏幕上
	- performTraversals 会依次调用 performMeasure、performLayout、performDraw 三个方法，这三个方法分别完成顶级 View 的 measure、layout 和 draw 这三大流程，其中 performMeasure 会调用 measure 方法，在measure 方法中又会调用 onMeasure 方法，在 onMeasure 方法中对所有的子元素进行 measure 过程，这个时候 measure 流程就会从父容器传递到子元素中了，这样就完成了一次 measure 过程。接着子元素就会重复父容器的 measure 过程，如此反复就完成了整个 View 树的遍历，同理 perFormLayout 和 performDraw 的流程也是类似 

# 2018.5.5

SeekBar

- 默认边距问题

	```
	android:paddingEnd="0dp"
	android:paddingStart="0dp"
	``` 

- 自定义thumb，会滑出进度条，或显示不全的问题

	```
	android:thumbOffset="0dp"
	```
