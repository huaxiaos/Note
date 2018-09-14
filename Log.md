# 2018.9.14

`Activity生命周期`

场景：在使用某app时，锁屏，然后打开系统app（电话、相机等），再退出（国外版的MOTO，7.0设备，关屏亮屏也会触发）

此时会触发这个app的onResume方法

也就是说，退出系统app时，会短暂的唤起这个app，然后再锁屏

# 2018.8.24

判断app前后台的方法

- 使用ActivityLifecycleCallbacks
- RunningTask&RunningProcess高版本的系统已经弃用
- [https://github.com/wenmingvs/AndroidProcess](https://github.com/wenmingvs/AndroidProcess)

# 2018.8.10

`动态代理`

- 区别于静态代理，动态代理不需要创建多个代理类，只用一个代理类即可，而且代理Object可以动态创建
- 使用方法
	- 代理类需要实现InvocationHandler这个接口
	- 通过Proxy.newProxyInstance创建代理类的对象
- 应用
	- hook
		- 动态代理是hook的一种方式，例如可以hook像Clipboard这种系统服务
			- Service缓存在一个map中
			- 跨进程通讯的IBinder对象，也缓存在一个map中
			- 可以通过动态代理修改map中的对象

# 2018.8.8

北京奥运10周年纪念日~

`TCP&UDP`

TCP三次握手

- Client–发送带有SYN标志的数据包–Server
- Server–发送带有SYN/ACK标志的数据包–Client
	- 为什么还需要ACK？因为双方通信无误必须是两者互相发送信息都无误，传了SYN，证明发送方到接收方的通道没有问题；传ACK，可以保证接收方到发送方的通道没有问题 	
- Client–发送带有带有ACK标志的数据包–Server

UDP

- 不需要先建立连接
- 不需要给出任何确认
- 一般用于即时通信

`JVM栈`

- 栈中保存的是基本数据类型和堆中对象的引用
- 基本数据类型长度固定，不会动态增长
- 传值使用的是“传引用值”的方式，而不是直接传递原始对象
- 对象放在堆中，用来实现对象的动态增长

`Java引用类型`

虚引用（Phantom Reference）十分脆弱，它的唯一作用就是当其指向的对象被回收之后，自己被加入到引用队列，用作记录该引用指向的对象已被销毁

引用队列(Reference Queue)

- 一般情况下，一个对象标记为垃圾（并不代表回收了）后，会加入到引用队列
- 对于虚引用来说，它指向的对象会只有被回收后才会加入引用队列，所以可以用作记录该引用指向的对象是否回收

# 2018.8.7

`git`

.gitkeep 可以用来跟踪空文件夹

# 2018.8.6

`traceview`

```
Debug.startMethodTracing();
...
Debug.stopMethodTracing();
```

真实路径：/sdcard/Android/data/包名/files/dmtrace.trace

并不是传说中的/sdcard/dmtrace.trace，具体细节可以查看源码

# 2018.8.2

`编译问题`

如果遇到一直出现报红的引用，可以close project，然后重新open（不要从cache的列表中选）

`命令行`

adb pull /sdcard/test.trace  /Users/sunhuaxiao/Desktop/

# 2018.7.31

`Glide`

glide重置图片大小，直接用into(w,h)或者override不生效，需要额外添加fitCenter属性

可以参考这个[issue](https://github.com/bumptech/glide/issues/1051)，大意是说，如果不指定fitCenter属性，Glide将不会直接进行resize

# 2018.7.30

`windowSoftInputMode 应用场景`

值得一提的是，同样的设置，某些场景下，manifest设置不生效，需要代码动态设置

```
getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN);
```

- adjustUnspecified：当软键盘弹出时，系统自动指定窗口的调整模式，根据不同的情况会选择adjustResize或者adjustPan的一种
- adjustPan ： 当软键盘弹出时，会将主窗口的平移（translateY），来适应软键盘的显示，键盘会在EditText的下方
- adjustResize ： 当软键盘弹出时，会让布局重新绘制，这种一般适应于带有滑动性质的控制，让其向下滚动，然后适应软键盘的显示
- adjustNothing： 软键盘弹出时，主窗口不会做出任何反应

# 2018.7.27

`编译问题`

Could not find matching constructor for: org.gradle.util.Clock()

gradle版本号过高导致，降低至4.1及以下可以fix

`OnLowMemory & OnTrimMemory `

https://www.jianshu.com/p/a5712bdb2dfd

- OnLowMemory被回调时，已经没有后台进程；而onTrimMemory被回调时，还有后台进程。
- OnLowMemory是在最后一个后台进程被杀时调用，一般情况是low memory killer 杀进程后触发；而OnTrimMemory的触发更频繁，每次计算进程优先级时，只要满足条件，都会触发。
- 通过一键清理后，OnLowMemory不会被触发，而OnTrimMemory会被触发一次

`单例之静态内部类`

```
public class Singleton {  
    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    
    private Singleton (){}  
    
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE; 
    }  
}
```

# 2018.7.26

`对于集合或者对象的基本操作也应该封装一个util类`

- 集合是否合法
	- null
	- 空
	- out of index 
- 返回集合长度的方法
- 其他常用的操作，避免重复判断代码的书写

`MVP框架中对model的理解`

之前的理解是，model层包括对数据的获取（例如从网络获取数据）以及对数据的基本处理（例如JSON to XXXBean），另外，业务逻辑中对数据的改造由presenter来实现

有几点不太好的地方

1. model层不应直接负责数据的获取，数据的获取需要自己封装成一个功能模块对model层提供接口
2. presenter层不应该做model数据的改造，所有的改造都应该由model层自己处理，并对外开放接口

所以，model层应该包括对数据的全部处理，包括基础的类型转换，包括数据有效性的校验，也应该包括根据业务需要进行的改造；类似数据获取的逻辑，应该单独出一个功能模块

`编译问题`：gradle编译报错，error如下
`com.android.build.gradle.tasks.factory.AndroidJavaCompile.setDependencyCacheDir(Ljava/io/File;)V`
原因：Android gradle plugin版本太旧
解决方案：update `com.android.tools.build:gradle:2.2.3` from 2.2.3 to 2.3.3

# 2018.6.9

深度优先 & 广度优先

https://blog.csdn.net/dyy_gusi/article/details/46414677

WebView静音

mWebView.reload();

# 2018.6.4

热修复

- 底层替换方案（native）
	- 直接在Native层修改原有类，不会再次加载新类，不会重启
	- 不能够增减原有类的方法和字段 
	- AndFix
- Dex插桩方案
	- 插入到dexElements数组的最前面，利用类加载机制实现
	- 需要重启
	- Tinker
- Instant Run热插拔原理
	- 参考Instant Run原理，修改$change
	- 美团Robust

[参考链接](https://blog.csdn.net/itachi85/article/details/79522200)	

资源热修复

- 构造一个新的AssetManager，反射调用addAssetPath
- 构造一个package id为 0 x 66 的资源包，不会和目前的资源包冲突，直接加入到AssetManager中就可以用

# 2018.6.2

解决的难题

- 扫描速度优化
	1. 特殊格式文件的筛选优化（例如WEBP格式）
		- 优化前：判断是否是WEBP格式的文件，是通过C层走ffmepg进行判断，一张图片的耗时为40ms左右（时间与机型等外界条件有关）
		- 优化后：通过读取文件的头16个字节，来快速判断WEBP格式的文件，一张图片的耗时为4ms左右
	2. 线程池
		- 优化前：一个App整体是一个子线程来负责扫描，没有用到线程池，每次扫描Thread的额外创建，浪费了过多时间
		- 优化后：增加线程数，一个App目录，进行进一步分解，一级目录由单独的线程来负责（例如gifshow和data\com.gifshow），并增加了线程池，耗时减少了50%，需要注意线程安全的问题
	3. 增加实时扫描输出回调，体验优化
		- 优化前：微信这种大型App，内部文件很多（5000+的文件很常见），如果等5000个文件扫描完再展示给用户，耗时很长
		- 优化后：增加输出回调，并设置一个间隔，例如每超过100个文件回调一次，这样用户就不会等待太长时间，可以先操作已扫描的100个文件 
	4. 单例模式防止多次扫描
		- 优化前：扫描类只是一个普通的Java类，如果多次调用，会多次执行扫描
		- 优化后：将扫描类封装成单例，App内的组件共享同一份资源，不会重复执行扫描

- 文件读写优化
	- 实时保存项目名称、实时保存项目缓存的JSON串
		- 问题：EditText中通过addTextChangedListener来获取用户输入的项目名，如果每次发生变化都进行文件存取，则对I/O的操作会过于频繁，例如，用户输入1234，listener回调会有4次，显然，我们只需要保存最后一次1234即可
		- 解决方案：
			- Handler消息队里，每次更新时清空之前队列中的操作
			- Handler延迟执行
- Bitmap优化 

NestedScroll

[android NestedScroll嵌套滑动机制完全解析](https://blog.csdn.net/lmj121212/article/details/52974427)

过度绘制

屏幕上的某个像素在同一帧的时间内被绘制了多次

- 移除默认的 Window 背景
- 移除不必要的背景，如子View和父View背景一致
- 写合理且高效的布局
	- 减少View层级
	- ViewStub
	- Merge标签  

稳定性

- 指标
	- 业务稳定性
		- 具体业务逻辑的稳定性，题库，剪辑操作等等 
	- 基础能力稳定性
		- Push到达率，如对接厂商push等
	- 性能稳定性
		- crash
		- 内存OOM
		- 卡顿、FPS
		- 启动时间
- 工具
	- Bugly、友盟、听云
	- Monkey
	- 自定义的测试脚本
	- TestIn兼容性
	- 阿里百川反馈
- 日志
	- 本地Log
	- Log收集页面
	- 核心逻辑处异常打点
		- 第三方埋点
		- 上传至服务端收集分析，如模考时间、自动交卷等异常问题
- 发布
	- A/B Test
	- 灰度发布、逐步放量发布
	- 热修复       

[2000万日订单背后：美团外卖客户端高可用建设体系](https://mp.weixin.qq.com/s/VCR4hdmE0ZsxjZsM_6g19w)

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
