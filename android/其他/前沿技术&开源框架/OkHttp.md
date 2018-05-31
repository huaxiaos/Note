# 基本结构

- OkHttpClient
	- Call
		- newCall
		- RealCall
	- Dispatcher
	- Interceptors

# Interceptors

拦截器，责任链模式，好处是将请求的发送和处理分开，并且可以动态添加中间的处理方实现对请求的处理等等，例如获取下载进度、统一添加请求头、统一做加密解密等等

拦截器按顺序执行，内部采用List实现

# Dispatcher

在Okhttp中Dispatcher负责将每一次Requst进行分发，压栈到自己的线程池，并通过调用者自己不同的方式进行异步和同步处理

OkHttpClient → Request → Call → Interceptors → Dispatcher

# Cache

OkHttpClient的Build方法中，可以自定义Cache，实现缓存

# 参考链接

https://yuqirong.me/2017/06/25/%E4%B8%80%E8%B5%B7%E6%9D%A5%E5%86%99OkHttp%E7%9A%84%E6%8B%A6%E6%88%AA%E5%99%A8/