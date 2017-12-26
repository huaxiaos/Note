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