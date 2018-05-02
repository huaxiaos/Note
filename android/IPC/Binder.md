# Binder应用场景

- Activity间传递对象需要序列化
- Activity启动流程
- 四大组件底层的通信
- AIDL内部的实现原理
- 插件化

# Binder优点

- 性能，只需要一次数据拷贝，性能上仅次于共享内存
- 稳定性，基于C/S架构，职责明确、架构清晰，因此稳定性好
- 安全性，为每个APP分配UID，进程的UID是鉴别进程身份的重要标志

# 传统IPC通信过程

1. 数据发送方，发送数据，通过系统调用copy_from_user()，将数据copy到内核缓存区
2. 数据接收方，通过系统调用copy_to_user()，将数据从内核缓存区copy到用户空间，接收数据

# Binder通信过程

1. Binder驱动在内核空间创建一个数据接收缓存区
2. 建立映射关系
	1. 内核缓存区、内核中数据接收缓存区
	2. 内核中数据接收缓存区、接收进程用户空间地址
3. 数据发送方，发送数据，通过系统调用copy_from_user()，将数据copy到内核缓存区
4. 数据接收方，通过映射关系获取到数据

# 从访问网页的流程来看Binder的通信流程

## 访问google的流程

1. 个人电脑，发起对www.google.com的请求
2. 从DNS服务器，得到IP地址的返回，例如192.168.245.123
3. 个人电脑，向IP为192.168.245.123的服务器发起请求
4. 服务器返回请求的数据

## Binder通信流程

1. Server进程，通过Binder驱动，向ServiceManager中注册服务
2. Client进程，查询Server进程的地址
3. Client进程，通过Binder驱动，得到ServiceManager返回的Server进程地址
4. Client进程，使用服务
5. Server进程，向Client进程提供服务，并返回数据

# Binder通信中的代理模式

当 Client 进程想要获取 Server 进程中的 object 时，驱动并不会真的把 object 返回给 Client，而是返回了一个跟 object 看起来一模一样的代理对象 objectProxy，这个 objectProxy 具有和 object 一摸一样的方法，但是这些方法并没有 Server 进程中 object 对象那些方法的能力，这些方法只需要把把请求参数交给驱动即可。对于 Client 进程来说和直接调用 object 中的方法是一样的

# 参考资料

- [https://juejin.im/post/5acccf845188255c3201100f](https://juejin.im/post/5acccf845188255c3201100f)