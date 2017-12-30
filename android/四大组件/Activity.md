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

# 简要执行流程

- Activity 
	- startActivity
	- startActivityForResult
- Instrumentation
	- execStartActivity
- ApplicationThread
	- scheduleLaunchActivity 
- ActivityThread
	- H
	- handleLaunchActivity
	- performLaunchActivity 

# 参考链接

- http://blog.csdn.net/singwhatiwanna/article/details/18154335