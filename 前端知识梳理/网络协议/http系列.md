# HTTP系列
![[htto系列协议栈.png]]
## 一.http的报文
### 1.报文结构
- 起始行（请求行，状态行）
- 报文头 （请求头，响应头）
- 报文体 （请求体，响应体）

注意：报文头部与报文体中间通过一个空行隔开。如果头部中有一个CRLF导致出现换行，那么下面的头部内容也会被认为是报文体

![[报文结构.png]]
![[报文结构2.png]]
### 2.请求行
- 请求的方法：GET/POST
- 请求目标: URI,资源的定位
- 协议版本：http/1.1
```js
GET / HTTP/1.1
```

三个部分通常使用空格（space）来分隔，最后要用 CRLF 换行表示结束。

![[请求行.png]]
### 3.状态行
- 协议版本：http/1.1
- 响应的状态码： 200
- 原因：状态码的补充，更详细的解释

``` js
HTTP/1.1 200 OK
```
![[状态行.png]]
### 4.头部字段
- key-value 的形式：Host: 127.0.0.1
- 字段名不区分大小写，中间不能有空格
- 字段名后必须是“:”, ":"后面可以是多个空格，一般是一个，空格无意义节约传输成本
- 字段没有顺序，字段名一般也不能重复复。除非这个字段本身的语义允许，例如 Set-Cookie
- 常用头部字段
	1.  通用字段：在请求头和响应头里都可以出现；
		1.  Cache-Control 缓存控制
	2.  请求字段：仅能出现在请求头里，进一步说明请求信息或者额外的附加条件；
		1.  accept：接收的文档类型
		2.  User-Agent：用户终端信息
	3.  响应字段：仅能出现在响应头里，补充说明响应报文的信息；
		1.  Access-Control-Allow-Origin：设置允许访问的源
		2.  Location: 重定向的字段
		3.  content-type
	4.  实体字段：它实际上属于通用字段，但专门描述 body 的额外信息。

### 5.请求的方法
**可以理解为对资源操作的动作**
-	GET: 获取资源，可以理解为读取或者下载数据
-	HEAD: 获取资源的元信息
-	POST: 向资源提交数据，相当于写入或上传数据
-	PUT: 类似 POST
-	DELETE：删除资源
-	OPTIONS：列出可对资源实行的方法
-	CONNECT： 建立特殊的连接隧道
-	TRACE： 追踪请求 - 响应的传输路径

### 6.状态码
- 1xx：目前是协议处理的中间状态，还需要后续的操作
- 2xx：表示成功
	- 200
	- 204：与200一样，但是没有响应体返回，空响应
	- 206：范围请求，分块下载，断点续传的基础
		- 伴随着头字段“**Content-Range**”，表示响应报文里 body 数据的具体范围，供客户端确认
- 3xx：重定向
	- 301：永久重定向
	- 302：临时重定向
	- 304：使用缓存
- 4xx： 客户端错误
	- 400
	- 403: 不允许访问
	- 404: 资源找不到
- 5xx：服务端错误
	- 501
	- 502

### 7.重定向
- 关键：响应头返回字段：Location: url
- 问题：
	1、重定向多一次跳转，导致资源浪费
	2、重定向如果策略有问题，可能导致循环跳转。A=>B=>A=>B
### 8.Cookie
由于http的无状态的特点导致身份验证等问题，Cookie可以存储信息，随着http请求带给服务器

- cookie的大小在4k左右，不存储大信息
- 作用：
	- 保存用户的登录信息
	- 广告跟踪
- cookie的对于字段
	- 请求头：Cookie
	- 响应头： Set-Cookie，key=value的形式
- 保护cookie
	- 设置有效时间: Max-Age，过期时间: Expires
	- 设置作用域：
		Domain和Path指定了 Cookie 所属的域名和路径，浏览器在发送 Cookie 前会从 URI 中提取出 host 和 path 部分，对比 Cookie 的属性。如果不满足条件，就不会在请求头里发送 Cookie。
		
	- 设置可读性，不能通过js访问，cookie只作用于http请求，HttpOnly
	- 限制第三方域名访问，SameSite=Strict
	- 仅能用 HTTPS传输 SameSite=Secure
## 二.http的特点
### 1.特点
- 简单，可扩展性强
	- 可以扩展头部信息
	- 报文体的信息也可以自定义
- 可靠
	- 基于TCP/IP，数据有序可靠传输
- 无状态
	- 每个请求都是独立的无关联，不要求客户端，服务端记录信息
- 应用广泛
	- 相比于SSH登录，FTP文件传输等协议，应用更广
- 请求应答模式：
	- 客服端请求，服务端响应
### 2.缺点
- 无状态
	- 请求间独立，导致需要每次都验证信息等，cookie可以解决
- 明文传输
	- 请求信息明文显示
- 不安全
	- 无法保证通讯双方的身份以及内容是否被篡改
- 性能
	- 基于TCP/IP，为了保证传输可靠，所以会有队头阻塞
### 3.HTTP 数据类型和语言类型
- MIME type：数据类型
	- Accept：希望接受的数据类型
	``` js
	Accept: text/html,application/xml,image/webp,image/png
	```
- 压缩方式： 数据压缩
	- Accept-Encoding
	``` js
	Accept-Encoding: gzip, deflate, br
	Content-Encoding: gzip
	```
- 语言类型： 
	- Accept-Language
	```js
	Accept-Language: zh-CN, zh, en
	```







## 三.http1.0
### 连接形式-短链接
每次连接都要重新TCP连接和关闭，从而浪费性能
### 示意图
![[http-短链接.png]]
## 四.http1.1
### 连接形式-长连接
“**Connection: keep-alive**”
为了降低连接的成本，所以多个来连接服用一个TCP通道，不用每次都重来。
### 示意图
![[http-长连接.png]]
### 问题
- 建立长连接，服务端需要维护多个长连接，可能出现连接只连不用从而浪费资源
	- 设置超时关闭，“keepalive\_timeout 60”
- 理论上连接越多，响应越快，但是服务器可能负担不了。
	- 服务器设置最大并行连接数。keepalive\_requests 5
	- 浏览器对域名同时访问的并行连接数（6-8）。通过域名分片可以突破限制
-  http层队头阻塞，请求-应答模式。原因：
	- http是请求-应答模式，一个请求必须等上个请求回来。
	- TCP保证数据传输可靠，丢包时会重新发送，所以http请求可能会阻塞
- TCP 队头阻塞

## 五.http/2
### http/2的特点
#### 兼容http/1.1
-	请求的方法
-	请求的URL
-	状态码，头字段
#### 头部压缩
- HPACK算法压缩，字典
- 静态字典：常用的字段，在客户端，服务端固定字典查询
- 动态字典：动态变化，如果一个自定义的字段传输使用多次，也会根据需要存入字典
#### 安全性
- 加密传输，基于SSL/TSL
- HTTP/2 协议定义了两个字符串标识符：“h2”表示加密的 HTTP/2，“h2c”表示明文的 HTTP/2
#### 多路复用
一个TCP通道可以被多个请求同时使用，且数据是二进制是数据流的形式传输。这样就解决了http层的队头阻塞（一个请求必须等上一个请求完成才可以开始）
- 消息不再是“Header+Body”的形式，而是分散为多个二进制“帧”
- 不同的数据流有不同的id
- 同一条数据流的每帧数据都带有序号，所以无需按顺序收发
- 在流上发送“RST\_STREAM”帧可以随时终止流，取消接收或发送
- 流可以设置优先级，让服务器优先处理，比如先传 HTML/CSS，后传图片，优化用户体验；
- 流之间没有固定关系，彼此独立，但流内部的帧是有严格顺序的
##### 服务端推送
客户端和服务端都可以创建流并且发送数据

### http2依然存在队头阻塞？
http2通过多路复用解决的http层的队头阻塞，但是TCP层的队头阻塞依然存在

TCP队头阻塞：

TCP需要保证数据的可靠传输，所以当一个包未接收到时，其他的数据包都会存在缓存中等待丢失包的重传。当缓存满了，导致后续后续来的数据，无法被处理，依旧形成阻塞

## 六.http/3
解决队头阻塞，握手更块（1个RTT）
![[http3.png]]
1.  HTTP/3 基于 QUIC 协议，完全解决了“队头阻塞”问题，弱网环境下的表现会优于 HTTP/2；
2.  QUIC 是一个新的传输层协议，建立在 UDP 之上，实现了可靠传输；
3.  QUIC 内含了 TLS1.3，只能加密通信，支持 0-RTT 快速建连；
4.  QUIC 的连接使用“不透明”的连接 ID，不绑定在“IP 地址 + 端口”上，支持“连接迁移”；
5.  QUIC 的流与 HTTP/2 的流很相似，但分为双向流和单向流；
6.  HTTP/3 没有指定默认端口号，需要用 HTTP/2 的扩展帧“Alt-Svc”来发现。

