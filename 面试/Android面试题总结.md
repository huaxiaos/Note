# 为什么Volley不适合大量数据的POST和大文件下载，而OkHttp适合？

> 裂变科技

Volley特别适合数据量小，通信量大的客户端

volley中为了提高请求处理的速度，采用了ByteArrayPool进行内存中的数据存储的，如果下载大量的数据，这个存储空间就会溢出，所以不适合大量的数据，但是由于他的这个存储空间是内存中分配的，当存储的时候优是从ByteArrayPool中取出一块已经分配的内存区域, 不必每次存数据都要进行内存分配，而是先查找缓冲池中有无适合的内存区域，如果有，直接拿来用，从而减少内存分配的次数 ，所以他比较适合大量的数据量少的网络数据交互情况

ByteArrayPool的作用，主要是为了减少GC和内存分配

Volley的线程池是基于数组实现的（默认大小为4），一旦大数据上传或者下载长时间占用了线程资源，后续所有的请求都会被阻塞

[http://www.voidcn.com/article/p-klgugreu-dw.html](http://www.voidcn.com/article/p-klgugreu-dw.html)

对于OkHttp，缓存的实现是严格按照http协议来设计的，没有额外设计一个类似ByteArrayPool的缓存池，所以不会出现OOM的问题