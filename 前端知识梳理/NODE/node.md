# Node
## 一.简介
### 1.异步I/O
    文件读取
    处理网络请求
### 2.单线程
#### 优点
    不用像多线程那样注意状态同步的问题,没有死锁
    没有线程上下文切换所带来的的性能的开销
#### 弱点
    无法利用多核CPU
    错误会引起整个应用退出,应用的健壮性值得考验
    大量的计算占用CPU导致无法继续调用异步I/O
### 3.应用场景
#### I/O密集型
    node面向网络擅长并行I/O
    node利用事件循环的处理能力, 而不是为每个请求起一个线程,资源占用少
#### CPU密集型业务?
    node的异步I/O解决了在单线程上CPU和I/O之间阻塞无法重叠利用的问题
    如果是纯计算的场景,根本没有I/O,此类场景应该用多线程
    
    node通过以下两种方式充分利用CPU
        1. 通过C++扩展方式高效的利用CPU
        2. 通过子进程的方式,起一个常驻进程用于计算,然后通过进程间通信传递结果,将计算与I/O分离
    
#### 分布式应用
    node高效利用并行I/O的过程,也是高效利用数据库的过程.
    对于node是一个I/O调用,对于数据库确是一次复杂的计算,所以可以充分的压缩硬件资源

## 二.模块机制
### CommonJS解决的问题
    没有模块系统
    标准库较少
    没有标准接口
    缺乏包管理系统
### 1.CommonJS与node
    浏览器: BOM + DOM (W3C) + ECMAScirpt
                
    node: ECMAScirpt + CommonJS
    
### 2. CommonJS规范
    CommonJS对模块的定义主要分为: 模块引用,模块定义,模块标识
#### 模块引用
    
```
var math = require('math')
```
    require方法接受模块标识,以此引入一个模块的API到当前上下文

#### 模块的定义
    模块的上下文提供require方法引入模块
    提供exports对象导出模块的方法,变量
    模块中有一个module对象,代表模块本身
    exports是module的属性
    在node中一个文件就是一个模块
    
#### 模块标识
    传递给require的参数
    
#### 模块定义的作用
    将类聚的方法和变量等限定在私有的作用域中,同时支持导入和导出功能以顺畅的
    连接上下游依赖
    a                         b
     module                    module
      require    <--------      exports
      
      exports
### 3.模块的实现
#### 模块的实现
        路径分析
        文件定位
        编译执行
#### 模块的分类
    核心模块
    -----------------------
    核心模块在node源码编译时,已经编译成了二进制文件
    node进程启动时,核心模块直接加载进内存中
    所以引用核心模块时,没有忽略文件定位及编译执行
    路劲分析又是优先判断,所以加载速度最快
    
    文件模块
    -------------------------------
    文件模块是在运行时动态加载,需要完整的路径分析,文件定位
    编译执行,所以加载速度比核心模块慢

    无论是核心模块还是文件模块,第二次加载时,优先使用缓存
    不过核心模块的缓存检查优先于文件模块
    
#### 路径分析
    require(模块标志),基于模块标志查找
    检查缓存
    核心模块
    文件模块
        逐级查找node_modules
#### 文件定位
    文件没有扩展名,会逐级补充 .js, .json, .node尝试
    带上文件扩展名,速度会快些
#### 模块编译
    .js: 通过fs模块同步读取文件后编译执行
    .json: 通过fs模块同步读取后,用JSON.parse返回执行结果
    .node: 属于C/C++编写的扩展模块,通过dlopen()加载编译
    其他文件类型: 按.js执行
    --------------------
    编译成功的模块会将文件路径作为索引缓存在Module_cache
    对象上,提高二次引入的性能
    -------------------
    文件的编译方法会复赋值在require()的extension属性上
    
#### JS模块编译
    js文件模块会被包装成必报函数
    并且将exports,require,module,__filename,__dirname
    作为形参传入
    返回一个具体的function对象
    从而隔离了作用域,外面只能调用exports上的属性和方法
    ----------------
    exports = {} 是不行的,直接赋值形参是改变引用并不会改变外界
    
    所以
    exports.aa = {} 
    
    或者 exports是module上的一个属性
    module.exports = {}

#### 模块调用栈
    内建模块(C/C++) ---> 核心模块 (js) ---> 文件模块
    ----------------
    libuv库:实现跨平台的封装,从而调用不同平台的底层操作
    (事件循环,底层操作等)
    
## 三.异步I/O
### 1.线程与进程
#### 多线程
    多线程的代价创建线程和切换线程上下文的开销比较大
    状态锁及状态同步等
    但是可以提高多核CPU的利用率
#### node解决方式
    利用单线程远离多线程的死锁,状态同步问题
    利用异步I/O让单线程远离阻塞
    利用子进程提高CPU和I/O的利用率
### 2.非阻塞I/O
#### 阻塞与阻塞I/O
    操作系统内核对I/O只有两种: 阻塞与非阻塞
    阻塞I/O: CPU等待I/O,CPU的处理能力得不到充分利用
    
    非阻塞: 不带数据直接返回,为了获取数据,需要重复check --- 轮训
    让cpu处于判断状态,造成cpu的浪费

#### 轮训的进化
    read: 重复检查I/O状态
    select: 通过文件描述符上的事件状态,进行判断
    poll: 
    epoll: 轮训的时候未检测到I/O事件蒋会休眠,直到事件发生将其唤起
    
    -----------------------------
    轮训可以满足非阻塞I/O获取完整数据,但仍然需要等待I/O返回
    遍历文件操作符,或者休眠,都会浪费CPU

### 3.异步I/O
#### 实现原理
    多线程
    利用线程池来玩成非阻塞I/O加轮训的技术完成数据获取
    主线程进行计算处理
    通过线程之前的通信将I/O获得数据进行传递
#### I/O
    文件,硬件,套接字等几乎所有的计算机资源都被抽象为文件
    所以网络请求,套接字同样适用阻塞与非阻塞的情况
#### 单线程?
    node是单线程的,指的是js执行在单线程中而已
    在node中,完成I/O任务的都是有线程池
    除了JS代码,其他的I/O都是可以并行执行
### 4.node异步I/O
#### 事件循环
    node事件循环会不断的执行tick --- 单个循环
    每次单个循环都有一个或者多个观察者
    --------------------------
    事件循环是生产者/消费者模型,
    异步I/O,网络请求都是事件的生产者,不断的提供事件到观察者
    事件循环从观察者那里取出事件并处理
    
#### 请求对象
    js发起内调用到内核执行完I/O操作的过程中会产生请求对象
    -------------------------
    请求对象上封装了传入的参数,回调方法
    然后将请求对象推入线程池,等待执行
    I/O在线程池中等待,不管是否是阻塞I/O都不会影响JS线程执行,从而达到异步
    I/O回调的结果
    -----------------------
    请求对象保存所有的状态,包括送入线程池,等待执行
    及I/O操作完毕后的回调处理

#### 执行回调
    I/O观察者的回调函数就是取出请求对象上的结果及执行方法
    然后执行调用
    
#### 总结
    事件循环,观察者,请求对象,线程池构成了node异步I/O模型
### 四.非I/O的异步API
#### 1.定时器
    实现与异步I/O类似,只是不需要线程池的参与
    创建时会插入到定时器观察者内部的一个红黑树
    
#### 2.process.nextTick()
    比setTimeout(fn,0)轻量
    每个tick间执行
    idle观察者
#### 3.setImmediate()
    process.nextTick 优先级 高于setImmediate
    check观察者
    ----优先级-----
    idle观察者 > I/O观察者 > check观察者
#### 4.执行
    process.nextTick 的回调函数存储在一个数组中
    setImmediate保存在链表中
    
    每次循环会将process.nextTick的回调函数全部执行
    setImmediate只执行一个

### 五.事件驱动,高性能服务器
#### 1.网络请求
    对于网络套接字的处理,node也是应用了异步I/O
    网络套接字上侦听到的请求都会形成事件交给I/O观察者
#### 2.高效服务器
    nginx,node都是基于事件驱动
## 四.内存控制
### 一.V8的内存限制
    node只能使用部分内存
    64位系统: 1.4G
    32系统: 0.7G
### 二.V8对象分配
    声明变量并赋值的时候,所使用的的对象内存就放在堆中
    如果已申请的堆内存不够则继续申请,直到超过V8限制
### 三.垃圾回收
#### 1.内存分布
    内存分为新生代,老生代
    新生代: 存活时间较短
    老生代: 存活时间较长或者常驻对象
    ---------------------
    V8堆大小 = 新生代 + 老生代
#### 2.新生代对象
    新生代对象由两个reserved_semispace_size_组成
    最大空间: 64位系统 32MB   32系统 16MB
    ------------------------------
    通过复制的方式实现垃圾算法
    将堆内存一分为二
    from空间: 使用中
    to空间: 空闲中
    -----------------
    进行垃圾回收时:检查from中的对象,将存活的对象复制到to空间
    其他的对象占用的空间释放
    复制完成后,to和from的身份互换
    -------------
    牺牲空间换取时间,新生代对象的生命周期短适合这个算法
#### 3.对象晋升
    一个新生代对象进过多次复制后仍然存活,那么会认为是生命周期较长的对象,
    从而引动到老生代对象中,
    或者当复制时,To空间的使用占比已经超过25%
#### 4.老生代对象
    标记阶段遍历所有老生代对象,并标记所有存活的对象
    清理死亡的对象
    -----------------------
    会造成内存的不连续
    对象标记死亡后,在清理的过程中,将活着的对象向一端移动
    然后直接清理边际外的空间,这样内存就连续了
    ---------------------
    为了避免垃圾回收时,程序全暂停,V8会进行增量标记,而不是一次性标记完

### 四.内存
#### 1.主动释放内存
    赋值为null,或者undefind
#### 2.堆外内存
    堆中的内存用量,总是小于进程中的常驻内存,这说明node的内存并非都是通过V8分配的
    那些不通过V8分配的内存称为对外内存
    比如:Buffer
#### 3.内存泄漏
    缓存
    -------------------------
    谨慎的将缓存存在内存中,可以使用redis解决
    好处: 进程间可以共享缓存,垃圾回收更高效
    
    
    队列消费不及时
    -----------------
    比如:日志吸入数据库
    解决:监控队列长度,一旦堆积马上报警
    
    作用域未释放

## 五.Buffer
### 1.简介
    Buffer是一个想array的对象,主要用于操作字节
    性能相关的部分由C++实现,非性能部分由js实现
    Buffer的内存不是通过V8实现的,属于堆外内存
### 2.Buffer
#### Buffer对象
    buffer对象类似于数组,他的元素为16进制的两位数
    即0到255的数值
    如果小于0,那么依次加上256,直到达到0-255
    如果小数,那么舍弃小数
#### 内存分配
    Buffer是有C++层面实现内存申请,在JS层面实现内存分配
    -----------------
    sleb分配机制
    slab是一块预先申请好的固定的内存区域,有三种状态
    full:完全分配
    partial: 部分分配
    empty: 没有被分配
    每个slab是8kb,作为内存分配的单元
    ----------------------------
    对于小而频繁的BUffer操作,采用slab机制进行预先申请和分配
    对于大块的Buffer而言,则直接使用C++层提供内存,则无需细腻的分配操作
### 3.Buffer转换
#### 字符串转Buffer
    new Buffer(str, [encoding])
    encoding不传时,默认以UFT-8进行转码和存储
#### Buffer转字符串
    buf.toString()
#### 支持的编码
    Buffer.isRncoding()判断编码是否支持转换
#### Buffer拼接
    一旦有宽字节,就会出现乱码字符,如汉字
    中文在UTF-8中占3个字节,所以当前后截断时,会出现乱码
    ----------------------------------------
    1.setEncoding()
    re.setEncoding('utf8')
    
    2.Buffer.concat
```
var chunks = []
var size = 0
res.on('data',function(chunk){
    chunks.push(chunk);
    size+=chunk.length
})
res.on('end',function(){
    var buf = Buffer.concat(chunks,size)
    var stt = iconv.decode(buf, 'utf8')
})
```
### 4.Buffer性能
#### 动静分离
    预先将静态的内容转为buffer对象,可以有效的减少cpu的重复利用
#### 文件的读取
     读取大文件,highWaterMark越大,读取越快
## 六.进程
### 1.单线程的问题
    如何利用多核CPU
    如何保证进程的健壮性和翁定性
### 2.多进程构建
#### 子进程
    child_process模块
    child_process.fork()
    每个进程有独立的V8实例
#### 创建子进程
    spawn()
    exec()
    execFile()
    fork
### 3.进程间通信
#### 通信的方法
    send()
    on('message')
#### IPC通信的原理
    Node实现IPC通道的是管道
    1.父进程在实际创建子进程之前,会创建IPC通道并且监听
    然后才会真正的创建子进
    2.父进程将该IPC通道的的文件描述符通过环境变量告诉子进程
    3.子进程在启动时,根据文件描述符去连接IPC通道
    4.完成父子进程间的连接
### 4.句柄传递
#### 句柄
    句柄: 一种可以用来标志资源的引用,如socket对象,套接字等
    -----------------------------
    send方法不仅可以发送数据,也可以发送句柄(第二个可选参数)
#### 监听同一个端口
    主进程在接收到socket请求后,直接将socket发送给工作进程.
#### 句柄发送的的类型
    net.Socket TCP的套接字
    net.server TCP服务
    -------------------------------
    发送到IPC管道中的实际上是句柄文件描述符
    整个message对象会通过JSON.stringfy进行序列化
    所以发送的是字符床
    --------------------------------
    父进程
        stringfy()
        send(fd)
    IPC
        parse()
        got(fd)
    子进程
    通过message.type和文件描述符还原出一个对象
    -----------------------------
    Node之间只有消息传递,不会真正的传递对象
    
### 5.实现
#### 直接实现
    
``` js
master.js
var net = require('net')
var cp = require('child_process').fork('child.js')
var server = net.createServer()
server.on('connection', function(socket) {
    socket.end('master')
})
server.listen(80, function() {
    cp.send('server', server) // 发送给子进程
})

child.js
process.on('message', function(m, server) {
    if(m==='server') {
        server.on('connection', function(socket) {
            socket.end('child')
        })
    }
})
```
#### 主进程发送句柄后关闭监听

```js
master.js
var net = require('net')
var cp = require('child_process').fork('child.js')
var server = net.createServer()
server.listen(80, function() {
    cp.send('server', server) // 发送给子进程
    server.close() // 关闭服务器的监听
})

child.js

var http = require('http')
// createServer(listener) --- listener是connection的监听器
var server = http.createServer(function(req,res) {
    res.end('child')
})

process.on('message', function(m, tcp) {
    if(m==='server') {
        tcp.on('connection', function(socket) {
        // http 也是继承于eventemitter
        // 接收到请求后主动触发connection
            server.emit('connection', socket)
        })
    }
})

```
#### 端口共同监听的原理
    独立的进程中,TCP的socket套接字的文件描述符并不相同
    导致监听相同的端口会掏出异常
    通过IPC传递的句柄还原出来的文件描述符一样,所以可以监听统一个端口
    由于文件描述符在同一时间只能被某个进程使用,所以想服务端
    发请求时,只会有一个进程为该次请求服务
### 6.集群稳定
#### 自动重启 --- 一个进程退出后,重启一个新的
        
```js
master.js
var fork = requier('child_process').fork
var cpus = require('os').cpus()
var server = require('net').createServer()
server.listen(8080)
var workers = {}
var createWorker = function() {
    var worker = fork('child.js')
    // 子进程退出的时候重启一个子进程
    worker.on('exit', function() {
        delete workers[worker.pid]
        createWorker()
    })
    worker.send('server', server)
    workers[worker.pid] = worker
}

for(var i = 0; i< cpus - 1; i++) {
    createWorker()
}

// master退出 --- 所有子进程退出
process.on('exit', function() {
    for(var pid in workers) {
        workers[pid].kill()
    }
})

```
#### 子进程因为错误关闭,要先断开链接,再退出
    
``` js
child.js

var http = require('http')
// createServer(listener) --- listener是connection的监听器
var server = http.createServer(function(req,res) {
    res.end('child')
})
var worker
process.on('message', function(m, tcp) {
    if(m==='server') {
    worker = tcp
        tcp.on('connection', function(socket) {
        // http 也是继承于eventemitter
        // 接收到请求后主动触发connection
            server.emit('connection', socket)
        })
    }
})
// 监听无捕获的错误
process.on('uncaughtException', function(){
    // 先关闭链接-再退出
    worker.close(function(){
        process.exit(1)
    }
})
```
#### 等进程关闭后再重启一个进程,可能导致没有可用进程
    所以需要在关闭之前给一个自杀信号,master立即重启一个新进程
    
``` js
child.js
// 监听无捕获的错误
process.on('uncaughtException', function(err){
    // 异常退出 --- log
    Logger.error(err)
    // 自杀信号
    process.send({act: suicide})
    // 先关闭链接-再退出
    worker.close(function(){
        process.exit(1)
    }
    // 设置超时退出
    setTimeout(function(){
        process.exit(1)
    }, 5000)
})

master
var createWorker = function() {
    var worker = fork('child.js')
    worker.on('message', function(msg) {
        // 监听自杀信号
        if(msg.act === 'suicide') {
            createWorker()
        }
    })
    // 子进程退出的时候重启一个子进程
    worker.on('exit', function() {
        delete workers[worker.pid]
    })
    worker.send('server', server)
    workers[worker.pid] = worker
}

```
#### 限量重启
    如果出现极端情况.工作进程启动就法伤错误,那么重启是
    无意义的,所以要在重启时候需要判断时间间隔
    当出现频繁重启的时候需要添加日志及报警
    
#### 负载均衡
    Node默认的机制是抢占是,即空闲的进程去抢占
    影响抢占的是CPU,而进程繁忙由cpu和I/O任务决定
    所以可能出现负载不均衡
    ------------
    轮叫调度策略
        主进程接受连接,将其依次分发给子进程
    -----------
    cluster模块中可以启用
#### 状态共享
    进程中不能存放太多数据,会影响垃圾回收,进而影响性能
    --------------------
    存在redis中,但是当数据变化时,需要去通知子进程
        1. 各个子进程去向第三方轮训
        2. 主动通知,其中一个轮训,变化后通知子进程
### cluster模块
#### 判断是否是主进程
    var cluster = require('cluster')
    cluster.isMaste
    cluster.fork()
    --------------
    cluster.setupMaster
#### 工作原理
    cluster是child_process和net模块的组合应用

    

    
    


    
    
    

        




    
    
    
    
    

    

    
    
    
      

    

    