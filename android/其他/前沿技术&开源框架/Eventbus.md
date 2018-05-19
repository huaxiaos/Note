# 原理

简单来说，是一个static的单例，成员变量中保存观察者，发送消息时，通知所有观察者

设计模式：发布订阅模式

使用了注解、反射

mSubcriberMap

- Key：EventType
- Value: CopyOnWriteArrayList

post方法

- 获取eventQueue
- 遍历执行Event
	1. postSingleEvent
	2. postSingleEventForEventType
	3. postToSubscription
	4. invokeSubscriber：subscription.subscriberMethod.method.invoke（反射）

# 使用场景

- ViewPager中相邻两个Fragment间的事件通知，例如，两个Fragment中显示的是音乐列表，点击其中一个Fragment的音乐，要同时改变另一个Fragment中音乐的展现样式
- 多个组件间的通信，例如，登陆注册完成后，需要通知其他控件执行某些事件
- 同一个事件的发生，要通知多个控件去处理

# 参考资料

- [https://juejin.im/post/5ad3e365f265da239c7bcf2b](https://juejin.im/post/5ad3e365f265da239c7bcf2b)