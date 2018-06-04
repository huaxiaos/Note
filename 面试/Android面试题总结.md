# RecyclerView & ListView 区别

> 美团

- 布局效果
	- RecyclerView，支持横向、纵向、Grid布局
	- ListView，纵向
- API使用
	- RecyclerView，复用 Item 的工作 Google 全帮你搞定，不再需要像 ListView 那样自己调用 setTag
	- ListView 自己实现
-  布局方法
	- ListView支持setEmptyView、HeadView、FootView、onItemClick
	- RecyclerView不支持，需要自定义实现
- 局部刷新
	- RecycleView支持
	- ListView不支持
- 动画效果
	- RecyclerView支持性更好
	- ListView需要完全自定义实现
- 嵌套滑动
	- RecyclerView支持，实现了NestedScrolling接口
	- ListView不支持
- 缓存
	- RecyclerView（四级缓存）
		- 屏幕内缓存
		- 屏幕外缓存，默认大小为2
		- 用户自定义的缓存
		- 缓存池（多个RecyclerView可以共用，例如ViewPager+多个list的场景） 
	- ListView（两级缓存）
		- 屏幕内
		- 屏幕外

[关于Recyclerview的缓存机制的理解](https://zhooker.github.io/2017/08/14/%E5%85%B3%E4%BA%8ERecyclerview%E7%9A%84%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6%E7%9A%84%E7%90%86%E8%A7%A3/) 

# onLayout

> 美团

onLayout的作用是用来确定子View的位置

自定义的ViewGroup必须要重写onLayout方法，通过getMeasuredWidth方法获取子View的宽度，然后根据需求，执行子View的layout方法，来实现定位

https://blog.csdn.net/dmk877/article/details/49632959

# ARGB_8888几位？几个字节？RGB相关

> 美团

- Bitmap.Config ARGB_4444：每个像素占四位，即A=4，R=4，G=4，B=4，那么一个像素点占4+4+4+4=16位 
- Bitmap.Config ARGB_8888：每个像素占四位，即A=8，R=8，G=8，B=8，那么一个像素点占8+8+8+8=32位
- Bitmap.Config RGB_565：每个像素占四位，即R=5，G=6，B=5，没有透明度，那么一个像素点占5+6+5=16位
- Bitmap.Config ALPHA_8：每个像素占四位，只有透明度，没有颜色。

一般情况下我们都是使用的ARGB_8888，由此可知它是最占内存的，因为一个像素占32位，8位=1字节，所以一个像素占4字节的内存。假设有一张480x800的图片，如果格式为ARGB_8888，那么将会占用1500KB的内存

# 如果有一个一直在执行的Thread，那么这个Thread和Service有什么区别？（不考虑ANR，不考虑内存泄漏）

> 美团

如果这个时候当你 start 一个 新的 Activity， 就没有办法在该 Activity 里面控制之前创建的 Thread

Thread没有bind这个功能

而Service是可以被控制的

# Java自动装箱&拆箱

> 陌陌

何时发生？

- 赋值
	- Integer iObject = 3 （装箱）
	- int iPrimitive = iObject （拆箱）
- 方法调用

	```
	public static Integer show(Integer iParam){
	   System.out.println("autoboxing example - method invocation i: " + iParam);
	   return iParam;
	}
	
	//autoboxing and unboxing in method invocation
	show(3); //autoboxing
	int result = show(3); //unboxing because return type of method is Integer
	``` 

[https://droidyue.com/blog/2015/04/07/autoboxing-and-autounboxing-in-java/](https://droidyue.com/blog/2015/04/07/autoboxing-and-autounboxing-in-java/)

# getChildFragmentManager & FragmentManager

> 美拍

- 需要管理相互独立的，并且隶属于Activity的Fragment使用FragmentManager
- 在Fragment中动态的添加Fragment要使用getChildFragmetManager来管理

# 子线程里面可以创建一个主线程相关的Handler么？

> 陌陌

可以

首先，需要从Handler的构造函数入手

```
public Handler() {
    this(null, false);
}

public Handler(Callback callback, boolean async) {
	...
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}

public Handler(Looper looper) {
    this(looper, null, false);
}

public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

可以看出Handler主要有两种构造函数

- Handler() 内部调用 Handler(Callback callback, boolean async) 
- Handler(Looper looper) 内部调用 Handler(Looper looper, Callback callback, boolean async)

对于第一个构造方法，mLooper = Looper.myLooper()，此时Looper是当前线程的Looper（存储在sThreadLocal里面）

对于第二个构造方法，从源码中可以看出，只要mLooper不是null即可，具体是子线程的Looper，还是主线程的Looper，没有要求，所以可以通过这个构造函数在子线程中创建主线程相关的Handler

# 一句话形容内存泄漏？举个例子？

> 陌陌

该被释放的对象没有释放，一直被某个或某些实例所持有，却不再被使用，导致 GC 不能回收

匿名内部类实现的Handler，会隐式引用外部的Activity，Activity在销毁的时候，会因为Handler的持有而造成 GC 回收失败

# i++ 原子性？分为几步？

> 陌陌

不具有原子性

i++，分为三步

1. 读取i的值
2. i+1
3. 写入新值

# Java 基本数据类型

> 陌陌

- byte：8位，1字节
- short：16位，2字节
- int：32位，4字节
- long：64位，8字节
- float：32位，4字节
- double：64位，8字节
- boolean
- char：16位，2字节

# Java构造方法&类初始化&显式调用&隐式调用

> 陌陌

```
static class A {
        static {
            System.out.println("A1");
        }
        {
            System.out.println("A2");
        }

        public A() {
            System.out.println("A3");
        }
    }

static class B extends A {
    static {
        System.out.println("B1");
    }
    {
        System.out.println("B2");
    }

    public B() {
        System.out.println("B3");
    }
}
```

new B(); 输出顺序

Right Answer：A1 B1 A2 A3 B2 B3

原则：（排名分先后）

1. 依次调用父类的static块、子类的static块（类初始化）
2. 依次调用父类的非static块、父类的构造器
3. 依次调用子类的非static块、子类的构造器
4. 第一步中的类初始化只会执行一次，多次new不会执行多次
5. new B()，会隐式调用父类的无参构造器（super）

# onSaveInstanceState和onRestoreInstanceState触发的时机

> 马蜂窝

onSaveInstanceState，原则是当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用

- HOME键
- 电源键关闭屏幕
- 从Activity A 中启动一个新的 Activity 时
- 屏幕方向切换

 onRestoreInstanceState，需要注意的是，onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用的，onRestoreInstanceState被调用的前提是，activity A“确实”被系统销毁了
 
 onRestoreInstanceState在onCreate方法之后执行，但onCreate能获取到onRestoreInstanceState中的bundle参数，从而可以做数据还原

# Fragment懒加载

> 马蜂窝

setUserVisibleHint

这个方法会在Fragment的默认生命周期之前触发，Fragment不可见时为false，可见时为true（TabLayout框架下，会触发多次）

可以在该方法中，Fragment可见时进行网络加载

# OkHttp、HttpClient、HttpURLConnection、Volley

> 马蜂窝

- HttpClient 是 Apache 的一个三方网络框架，5.0被废弃
- HttpURLConnection 是 Google 官方提供的网络库，底层也是用Socket来实现
- OkHttp 是 Square 开源的网络库，底层实现同一ip和端口的请求重用一个socket，降低网络连接时间，对https有更好的支持，I/O性能也更好
- Volley 是 Google 封装的网络框架，底层在android2.3以下系统使用httpclicent，在android2.3以上采用HttpUrlConnection，底层实现了ByteArrayPool缓存，适合频繁量小的网络操作

# Activity中的Flags

> 马蜂窝

- NEW_TASK，效果和singleTask相同 
- SINGLE_TOP，效果和singleTop相同
- CLEAR_TOP
	- 如果被启动Activity是SingleTask，那么栈上面的Activity出栈，执行该Activity的onNewIntent方法
	- 如果被启动Activity是standard，那么该Activity连同其上面的Activity会全部出栈，重现创建实例
- EXCLUDE_FROM_RECENTS，不会出现在历史Activity列表中  


# View的getX和getRawX的区别？getX会返回负值么？

> 马蜂窝

- getX相对于当前View左上角的坐标
- getRawX相对于屏幕左上角的坐标

如果手指滑出当前view的范围，那么 getX 或 getY 会返回负值

# Activity生命周期，A切换至B

> 马蜂窝

- A - onPause
- `B - onCreate`
- `B - onStart`
- `B - onResume`
- A - onStop

# 为什么serializable只需要实现接口就可以进行序列化？

> 马蜂窝

Java中ObjectOutputStream用于序列化，ObjectInputStream用于反序列化

从ObjectOutputStream的源码中可以得出答案，在序列化之前会进行类型判断，如果不是serializable、Enum、String等类型，就会抛异常

> 1. 在变量声明前加上transient关键字，可以阻止该变量被序列化
> 2. 在类中增加 writeObject 和 readObject 方法可以实现自定义序列化

```
// ObjectOutputStream#writeObject0

if (obj instanceof Class) {
    writeClass((Class) obj, unshared);
} else if (obj instanceof ObjectStreamClass) {
    writeClassDesc((ObjectStreamClass) obj, unshared);
// END Android-changed
} else if (obj instanceof String) {
    writeString((String) obj, unshared);
} else if (cl.isArray()) {
    writeArray(obj, desc, unshared);
} else if (obj instanceof Enum) {
    writeEnum((Enum<?>) obj, desc, unshared);
} else if (obj instanceof Serializable) {
    writeOrdinaryObject(obj, desc, unshared);
} else {
    if (extendedDebugInfo) {
        throw new NotSerializableException(
            cl.getName() + "\n" + debugInfoStack.toString());
    } else {
        throw new NotSerializableException(cl.getName());
    }
}
```

# 为什么Volley不适合大量数据的POST和大文件下载，而OkHttp适合？

> 裂变科技

1. 首先说下为什么Volley不适合
	1. Volley特别适合数据量小，通信量大的客户端
	2. volley中为了提高请求处理的速度，将数据存储缓存在ByteArrayPool（为了减少GC和内存分配），如果下载大量的数据，这个存储空间就会溢出OOM
	3. Volley的线程池是基于数组实现的（默认大小为4），一旦大数据上传或者下载长时间占用了线程资源，后续所有的请求都会被阻塞
2. 然后说下OkHttp为什么适合
	1. OkHttp给到用户的是一个InputStream，用户可以把读到的数据直接写到disk，这样一来，也就不占太多内存
	2. OkHttp的IO操作使用的是Okio，对下载所涉及到的IO操作进行了优化

[http://www.voidcn.com/article/p-klgugreu-dw.html](http://www.voidcn.com/article/p-klgugreu-dw.html)

# 如何实现动态设置观察者？比如广播发了一条消息，A B C 中 AB收到了消息，但C还没注册，如何接收消息？

> 裂变科技

EventBus 黏性事件

发布黏性事件的时候，将黏性事件存在一个stickyEvents的Map里面，当订阅者再次订阅时，如果发现是sticky事件，就会将缓存中的事件进行发布

需要注意的是，接受到sticky事件后，要注意移除黏性事件（具体是全部移除，还是移除单个，要看产品需求），否则会造成多次消费的问题

应用场景：比如登陆后执行播放音乐，音乐页面，对登陆的监听可能会没有完成注册，登陆动作就执行完毕了，如果是黏性事件就可以优雅的解决问题，只要注册了登陆监听，就能立即收到事件

# 如何设计一个下载库

> 快手、裂变科技

首先需要考虑需要给用户提供什么样的功能

- 支持超大文件下载，如果使用OkHttp，单个文件的限制是2G，超过2G的文件需要特殊处理（可以参考流利说的开源库中的处理方式）
- 断点续传
- 网络判断，断网、wifi等等
- 基本操作，开始、暂停、停止、clear等等
- 异常处理
- 缓存，数据持久化（DB，SP等等）
- 多线程处理，并发下载

然后对下载库进行整体架构设计

- connection：网络层处理
- database、cache：数据持久化
- download：执行下载action
- event、action：事件或action的定义
- exception：异常处理
- services、notification
- utils

# Integer占用多少个字节

> 裂变科技

32 位系统下，当使用 new Object() 时，JVM 将会分配 8（Mark Word+类型指针） 字节的空间，128 个 Object 对象将占用 1KB 的空间。

如果是 new Integer()，那么对象里还有一个 int 值，其占用 4 字节，这个对象也就是 8+4=12 字节，对齐（8字节的整数倍）后，该对象就是 16 字节

在 64 位系统及 64 位 JVM 下，开启指针压缩，那么头部存放 Class 指针的空间大小还是4字节，而 Mark Word 区域会变大，变成 8 字节

# Glide和Fresco的对比

|      | Glide | Fresco |
| :---: | :---: | :---: |
| layout | ImageView | SimpleDraweeView |
| 圆角 | 开发者自己实现 | RoundingParams参数 |
| 缓存 | 两级缓存：内存和磁盘缓存 | 三级缓存，分别是 Bitmap缓存，未解码图片缓存， 文件缓存 |
| 缓存图像大小 | 可以针对原始图像和结果图像做缓存 | 缓存原始图像 |
| 加载策略 | 只有占位图 | 先加载小尺寸图片，再加载大尺寸的 |
| 加载进度 | false | true |

Glide的优点显而易见，简单易用，使用绝大多数的App场景

对于Fresco而言主要有以下几个方面

- 最大的优势在于5.0以下(最低2.3)的bitmap加载。在5.0以下系统，Fresco将图片放到一个特别的内存区域(Ashmem区)
- 大大减少OOM（在更底层的Native层对OOM进行处理，图片将不再占用App的内存）
- 适用于需要高性能加载大量图片的场景

Fresco缓存策略：

Fresco使用三级缓存，已解码内存缓存；未解码内存缓存；磁盘缓存

- 第一级缓存就是保存bitmap，直接存的就是bitmap对象，5.0 以下，这些位于ashmem，5.0以上，直接位于java的heap上
- 第二级缓存保存在内存，但是没有解码，使用时需要解码，
- 第三级缓存就是保存在本地文件，同样文件也未解码，使用的时候要先解码啦！

在5.0以下，GC将会显著地引发界面卡顿。Fresco将图片放到一个特别的内存区域。当然，在图片不显示的时候，占用的内存会自动被释放。这会使得APP更加流畅，减少因图片内存占用而引发的OOM。

> Ashmem
> 
> Ashmem 是一种共享内存的机制，不同进程可以将同一段物理内存映射到各自的虚拟地址控制，从而实现共享。通过注册Cache Shrinker回收内存（不会触发GC）

# SSL握手是OkHttp实现的？还是底层实现的？

> 蚂蚁金服

对于https，在tcp三次握手后就会进行ssl的握手

核心方法在RealConnection类中的connectSocket方法，判断是否需要ssl，如果需要则执行connectTls，ssl握手执行的代码是

```
sslSocket.startHandshake();
```

startHandshake的实现在org.conscrypt.OpenSSLSocketImpl#startHandshake中，调用的是native函数NativeCrypto.SSL_do_handshake()

ssl握手完毕后，还会使用 HostnameVerifier 来验证 host 是否合法

应用实例：

如果服务端有自制https证书的需求，那么可以通过两个方法来实现

1. 信任所有证书，跳过Verifier，不使用默认的SSLSocketFactory，自定义TrustManager，信任所有证书
2. 信任自定义证书，以信任12306证书为例
	1. 将自签名证书，比如 12306 的 srca.cer，保存到 assets
	2. 读取自签名证书集合，保存到 KeyStore 中
	3. 使用 KeyStore 构建 X509TrustManager
	4. 使用 X509TrustManager 初始化 SSLContext
	5. 使用 SSLContext 创建 SSLSocketFactory

- https://juejin.im/entry/597f00ca6fb9a03c41455bf8
- https://blog.csdn.net/hello2mao/article/details/53201974