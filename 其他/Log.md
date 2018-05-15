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
