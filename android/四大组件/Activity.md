# Activity启动流程

Activity的完整启动流程非常复杂，但我们可以在Activity的入口处下断点，简要的看到Activity启动时的堆栈情况，如下图

![Activity启动时的堆栈](http://7xuvhf.com1.z0.glb.clouddn.com/FphHAfwJ5dwmTGbG_P3P_aUqbsF7)

简单来说，Activity启动的第一步是通过Zygote进行孵化，ZygoteInit执行一系列方法，创建出ActivityThread，之后启动ActivityThread中的Handler（H），依次通过handleLaunchActivity、performLaunchActivity通知Instrumentation，Instrumentation执行callActivityOnCreate方法通知Activity，Activity执行performCreate方法，通知具体的Activity，比如MainActivity，进而执行onCreate，时序图如下

![Activity启动流程时序图](http://7xuvhf.com1.z0.glb.clouddn.com/Fqawh0dLR3Rv1zOlnYDa6MzvUWLl)

Instrumentation其实是Android内部的一个测试框架，用来辅助Activity完成启动Activity的过程

# Activity启动模式

- standard 标准模式，每次都新建一个实例对象
- singleTop 如果在任务栈顶发现了相同的实例则重用，否则新建并压入栈顶
- singleTask 如果在任务栈中发现了相同的实例，将其上面的任务终止并移除，重用该实例。否则新建实例并入栈
- singleInstance 允许不同应用，进程线程等共用一个实例，无论从何应用调用该实例都重用

# Activity生命周期和Fragment生命周期的关系

Fragment生命周期和父Activity的生命周期是保持一致的，所以在程序设计的时候要考虑到两者的关系

例如，有一个场景，父Activity是singleTask类型，并且有一条通知点击效果是进入父Activity，那么会执行onNewIntent方法，接着会执行onResume方法，从而会引起Fragment执行onResume方法，如果Fragment的onResume方法中存在一些逻辑代码，可能会出现问题

可以在onResume方法中，增加isHidden()的判断，来做区分

# 生命周期

- 锁屏：onPause-onStop
- Alert弹出：onPause
- Home键：onPause-onStop

# Activity切换

- A - onCreate
- A - onStart
- A - onResume
- A - onPause
	- B - onCreate
	- B - onStart
	- B - onResume
- A - onStop

# 参考链接

- http://blog.csdn.net/singwhatiwanna/article/details/18154335