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

1. interceptors：用户自定义的拦截器
2. RetryAndFollowUpInterceptor：负责失败重试以及重定向
3. BridgeInterceptor：负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应的
4. CacheInterceptor：负责读取缓存直接返回、更新缓存的
5. ConnectInterceptor：负责和服务器建立连接的
6. networkInterceptors：用户自定义的网络拦截器
7. CallServerInterceptor：负责向服务器发送请求数据、从服务器读取响应数据

## ConnectInterceptor

主要工作是创建`HttpCodec`对象，`HttpCodec`是对HTTP协议的抽象，有`Http1Codec`（对应HTTP1.1）和`Http2Codec`（对应HTTP2）这两个实现类

其中会通过Okio对Socket的读写进行封装

## CallServerInterceptor

主要工作

1. 向服务器发送request header
	1. 如果有request body，就向服务器发送body
2. 读取response header，同时构造一个 Response 对象
	1. 如果有 response body，就在上一步的基础上加上 body 构造一个新的 Response 对象

# RealInterceptorChain

工作流程

- FirstChain.proceed
- FirstInterceptor.intercept
- ...proceed
- ...intercept
- LastChain.proceed
- LastInterceptor.intercept
- response to LastChain
- response to xxxChain
- response to FirstChain

# Dispatcher

在Okhttp中Dispatcher负责将每一次Requst进行分发，压栈到自己的线程池，并通过调用者自己不同的方式进行异步和同步处理

OkHttpClient → Request → Call → Interceptors → Dispatcher

# Cache

OkHttpClient的Build方法中，可以自定义Cache，实现缓存

# 参考链接

https://yuqirong.me/2017/06/25/%E4%B8%80%E8%B5%B7%E6%9D%A5%E5%86%99OkHttp%E7%9A%84%E6%8B%A6%E6%88%AA%E5%99%A8/