# http

# http 1.0

# http 1.1

默认支持长连接

# http 2.0

# 状态码

- 2XX 成功
	- 200 OK，表示从客户端发来的请求在服务器端被正确处理
	- 204 No content，表示请求成功，但响应报文不含实体的主体部分
	- 206 Partial Content，进行范围请求
- 3XX 重定向
	- 301 moved permanently，永久性重定向，表示资源已被分配了新的 URL
	- 302 found，临时性重定向，表示资源临时被分配了新的 URL
	- 303 see other，表示资源存在着另一个 URL，应使用 GET 方法丁香获取资源
	- 304 not modified，表示服务器允许访问资源，但因发生请求未满足条件的情况
	- 307 temporary redirect，临时重定向，和302含义相同
- 4XX 客户端错误
	- 400 bad request，请求报文存在语法错误
	- 401 unauthorized，表示发送的请求需要有通过 HTTP 认证的认证信息
	- 403 forbidden，表示对请求资源的访问被服务器拒绝
	- 404 not found，表示在服务器上没有找到请求的资源
- 5XX 服务器错误
	- 500 internal sever error，表示服务器端在执行请求时发生了错误
	- 503 service unavailable，表明服务器暂时处于超负载或正在停机维护，无法处理请求

# https

## SSL握手过程（非对称加密RSA+对称加密AES）

1. 客户端 A 访问服务器 B ，比如我们用浏览器打开一个网页 www.baidu.com ，这时，浏览器就是客户端 A ，百度的服务器就是服务器 B 了。这时候客户端 A 会生成一个随机数1，把随机数1 、自己支持的 SSL 版本号以及加密算法等这些信息告诉服务器 B 。
2. 服务器 B 知道这些信息后，然后确认一下双方的加密算法，然后服务端也生成一个随机数 2，并将随机数 2 和 CA 颁发给自己的证书(或者自定义的证书)一同返回给客户端 A （CA证书包含一对公钥和私钥）
3. 客户端 A 得到 CA 证书后，会去校验该 CA 证书的有效性，校验通过后，客户端生成一个随机数3 ，然后用证书中的公钥加密随机数3 并传输给服务端 B （客户端使用证书中的公钥加密）
4. 服务端 B 得到加密后的随机数3，然后利用私钥进行解密，得到真正的随机数3（服务端使用私钥解密）
5. 客户端 A 和服务端 B 都有随机数1、随机数2、随机数3，然后双方利用这三个随机数生成一个对话密钥。之后传输内容就是利用对话密钥来进行加解密了。这时就是利用了对称加密，一般用的都是 AES 算法

简要流程：

1. Client 请求 https 连接
2. Server 返回证书（CA或自定义证书，包含一对公钥和私钥）
3. Client 生成随机数，并使用证书中的公钥进行加密（生成对称密钥A）
4. Client 发送加密后的对称密钥
5. Server 通过证书中的私钥进行解密，然后使用对称密钥A来进行通信

# 参考资料

- [https://www.jianshu.com/p/52d86558ca57](https://www.jianshu.com/p/52d86558ca57)
- [https://juejin.im/post/5ad4094e6fb9a028d7011069](https://juejin.im/post/5ad4094e6fb9a028d7011069)
- [https://blog.csdn.net/clh604/article/details/22179907](https://blog.csdn.net/clh604/article/details/22179907)