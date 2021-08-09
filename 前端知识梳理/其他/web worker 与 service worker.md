# web worker 与 service worker
## 一.web worker
[参考](https://www.ruanyifeng.com/blog/2018/07/web-worker.html)
### 一.web worker的作用
- 为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程
- Worker线程在后台运行，不影响主线程
- Worker线程完成任务后可以将结果返回给主线程
### 二.特点
- **同源限制**：Worker 线程运行的脚本文件，必须与主线程的脚本文件同源
- **DOM 限制**： 无法访问网页的 DOM 对象，也无法使用`document`、`window`、`parent`这些对象。但是，Worker 线程可以`navigator`对象和`location`对象
- **通信联系**： 通过postMessage完成通信
- **脚本限制**： Worker 线程不能执行`alert()`方法和`confirm()`方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求
- **文件限制**：Worker 线程无法读取本地文件，即不能打开本机的文件系统（`file://`），它所加载的脚本，必须来自网络

### 三.基本用法
#### 1.主线程
创建Worker
```javascript
var worker = new Worker('work.js')
```
向Worker进程发送消息
```javascript
worker.postMessage('Hello World');
```
接收Worker的消息
```javascript
worker.onmessage = function (event) {
  console.log('Received message ' + event.data);
  doSomething();
}
```
Woreker发生错误时的消息
```javascript
worker.onerror(function (event) {
  console.log([
    'ERROR: Line ', e.lineno, ' in ', e.filename, ': ', e.message
  ].join(''));
});

// 或者
worker.addEventListener('error', function (event) {
  // ...
});
```

#### 2.Worker 线程
接收消息，发送消息
```javascript
self.addEventListener('message', function (e) {
  self.postMessage('You said: ' + e.data);
}, false);
```
加载脚本
```javascript
importScripts('script1.js', 'script2.js');
```
关闭 Worker
```javascript
// 主线程
worker.terminate();

// Worker 线程
self.close();
```
#### 3. 也可以通过二进制的形式，将数据传递给Worker进程
将一个 ArrayBuffer 对象从主应用转让到 Worker 中，原始的 ArrayBuffer 被清除并且无法使用。它包含的内容会(完整无差的)传递给 Worker 上下文

### 四.场景
1.  比较消耗cpu性能的计算，数学运算
2.  图像、影音等文件处理
3.  大量数据检索
    比如用户输入时，我们在后台检索答案，或者帮助用户联想，纠错等操作.
4.  耗时任务都丢到 webworker 解放我们的主线程。
5.  定时器



## 二.Service Worker
### 一.基本介绍
- 基于web worker，在web worker的基础上增加了离线缓存功能，并且开发者可以管理缓存内容
- 本质上充当 客户端和 服务端中的的代理服务器
- 可以离线访问，PWA的基础
- 可以访问cache和indexDB
- 事件驱动，具有生命周期
- 支持消息推送

### 二.生命周期
#### 1.注册
navigator.serviceWorker.register
```js
/* 判断当前浏览器是否支持serviceWorker */
    if ('serviceWorker' in navigator) {
        /* 当页面加载完成就创建一个serviceWorker */
        window.addEventListener('load', function () {
            /* 创建并指定对应的执行内容 */
            /* scope 参数是可选的，可以用来指定你想让 service worker 控制的内容的子目录。 在这个例子里，我们指定了 '/'，表示 根网域下的所有内容。这也是默认值。 */
            navigator.serviceWorker.register('./serviceWorker.js', {scope: './'})
                .then(function (registration) {
 
                    console.log('ServiceWorker registration successful with scope: ', registration.scope);
                })
                .catch(function (err) {
 
                    console.log('ServiceWorker registration failed: ', err);
                });
        });
    }
```

#### 2.安装，激活
在指定的处理程序`serviceWorker.js`中书写对应的安装及拦截逻辑
注册fetch事件，拦截全站的请求，存放到caches中
```js
/* 监听安装事件，install 事件一般是被用来设置你的浏览器的离线缓存逻辑 */
this.addEventListener('install', function (event) {
 	
    /* 通过这个方法可以防止缓存未完成，就关闭serviceWorker */
    event.waitUntil(
        /* 创建一个名叫V1的缓存版本 */
        caches.open('v1').then(function (cache) {
            /* 指定要缓存的内容，地址为相对于跟域名的访问路径 */
            return cache.addAll([
                './index.html'
            ]);
        })
    );
});

/* 注册fetch事件，拦截全站的请求 */
this.addEventListener('fetch', function(event) {
  event.respondWith(
    // magic goes here
      
      /* 在缓存中匹配对应请求资源直接返回 */
    caches.match(event.request)
  );
});
```

### 三.注意事项
- 不能访问DOM
- 完全异步，同步API（如XHR和localStorage）不能在service worker中使用
- 出于安全考量，Service workers只能由HTTPS承载
- 在Firefox浏览器的用户隐私模式，Service Worker不可用
- 其生命周期与页面无关

### 四.使用场景
- PWA
- 缓存
-  集中接收计算成本高的数据更新，比如地理位置和陀螺仪信息，这样多个页面就可以利用同一组数据
-   后台数据同步
-   响应来自其它源的资源请求