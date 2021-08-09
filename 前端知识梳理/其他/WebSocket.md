# WebSocket
## 一.简介
HTML5开始提供的一种浏览器与服务器进行全双工通讯的网络技术，属于应用层协议。它基于TCP传输协议，并复用HTTP的握手通道
### 1.优点
- 支持双向通信
- 更好的二进制支持。
- 基于TCP，复用http握手通道
- 较少的控制开销。连接创建后，ws客户端、服务端进行数据交换时，协议控制的数据包头部较小
- 支持扩展。ws协议定义了扩展，用户可以扩展协议，或者实现自定义的子协议。（比如支持自定义压缩算法等）

### 2.简单的例子
#### 服务端
``` js
var app = require('express')();
var server = require('http').Server(app);
var WebSocket = require('ws');
var wss = new WebSocket.Server({ port: 8080 });
wss.on('connection', function connection(ws) {
	 console.log('server: receive connection.');
	 ws.on('message', function incoming(message) {
	 	console.log('server: received: %s', message);
	 });
	ws.send('world');
});

app.get('/', function (req, res) {
 	res.sendfile(__dirname + '/index.html');
});
app.listen(3000);
```
### 客户端
``` js
var ws = new WebSocket('ws://localhost:8080');
ws.onopen = function () {
	 console.log('ws onopen');
	 ws.send('from client: hello');
};

ws.onmessage = function (e) {
	 console.log('ws onmessage');
	 console.log('from server: ' + e.data);
};
```

## 二..建立连接
### 1 客户端：申请协议升级
客户端发起协议升级请求。可以看到，采用的是标准的HTTP报文格式，且只支持`GET`方法

``` http
GET / HTTP/1.1
Host: localhost:8080
Origin: http://127.0.0.1:3000
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: w4v7O6xFTi36lq3RNcgctw==
```
-   `Connection: Upgrade`：表示要升级协议
-   `Upgrade: websocket`：表示要升级到websocket协议。
-   `Sec-WebSocket-Version: 13`：表示websocket的版本。如果服务端不支持该版本，需要返回一个`Sec-WebSocket-Version`header，里面包含服务端支持的版本号。
-   `Sec-WebSocket-Key`：与后面服务端响应首部的`Sec-WebSocket-Accept`是配套的，提供基本的防护，比如恶意的连接，或者无意的连接。

### 2.服务端：响应协议升级
服务端返回内容如下，状态代码`101`表示协议切换。到此完成协议升级，后续的数据交互都按照新的协议来。
``` http
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```
`Sec-WebSocket-Accept`根据客户端请求首部的`Sec-WebSocket-Key`计算出来。
1.  将`Sec-WebSocket-Key`跟`258EAFA5-E914-47DA-95CA-C5AB0DC85B11`拼接。
2.  通过SHA1计算出摘要，并转成base64字符串。

## 三.数据帧格式
### 1.数据通信格式
WebSocket客户端、服务端通信的最小单位是帧（frame），由1个或多个帧组成一条完整的消息（message）。

1.  发送端：将消息切割成多个帧，并发送给服务端；
2.  接收端：接收消息帧，并将关联的帧重新组装成完整的消息；

### 2.数据传递
WebSocket的每条消息可能被切分成多个数据帧。当WebSocket的接收方收到一个数据帧时，会根据`FIN`的值来判断，是否已经收到消息的最后一个数据帧。

FIN=1表示当前数据帧为消息的最后一个数据帧，此时接收方已经收到完整的消息，可以对消息进行处理。FIN=0，则接收方还需要继续监听接收其余的数据帧。

### 3.连接保持+心跳
-   发送方->接收方：ping
-   接收方->发送方：pong

``` js
ws.ping('', false, true);
```

### 4.Sec-WebSocket-Key/Accept的作用
`Sec-WebSocket-Key/Sec-WebSocket-Accept`在主要作用在于提供基础的防护，减少恶意连接、意外连接。



## 参考
[WebSocket](https://juejin.cn/post/6844903544978407431)