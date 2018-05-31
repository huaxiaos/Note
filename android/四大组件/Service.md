# 生命周期

## startService

- onCreate
- onStartCommand
- onDestroy

首次启动，会执行onCreate→onStartCommand方法；重复启动，只会执行onStartCommand方法

## bindService

- onCreate
- onBind
- onUnbind
- onDestroy

# bind

Activity中创建ServiceConnection，Service中实现onBind来建立连接

通过Binder实现数据传递

# Service&Thread

Service运行在主线程中（线程id和主线程id一致），如果执行过多的耗时操作，可能会引起ANR

Activity销毁时，Activity中生成的Thread会随之销毁，但Service不会销毁，可以通过重新建立绑定关系，进行继续的操作

Service和Activity可以是一对多的关系

# 远程Service

AIDL进行进程间通信

# IntentService

# 实际项目中的应用

App版本更新、文件扫描