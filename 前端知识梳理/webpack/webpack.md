# webpack
## 一.webpack的核心
实现前端项目的模块化
模块化发展的阶段
- 文件划分方式
	- 模块直接在全局工作，大量模块成员污染全局作用域；
	-   没有私有空间，所有模块内的成员都可以在模块外部被访问或者修改；
	-   一旦模块增多，容易产生命名冲突；
	-   无法管理模块与模块之间的依赖关系；
	-   在维护的过程中也很难分辨每个成员所属的模块。
- 命名空间方式
	- 只是解决了命名冲突的问题，但是其它问题依旧存在
- IIFE : 立即执行函数
	- 为模块提供私有空间。具体做法是将每个模块成员都放在一个立即执行函数所形成的私有作用域中，对于需要暴露给外部的成员，通过挂到全局对象上的方式实现
	- 解决了前面所提到的全局作用域污染和命名冲突的问题
- CommonJS，AMD，ES Module[[CommonJs,AMD]]
### 1.webpack的核心概念
- entry：入口文件
- module：模块，在webpack中一切皆是模块
-  Chunk：代码块，一个chunk由多个模块组合而成，用于代码合并与分割
- loader: 将各种模块的内容按需求转换输出
- output：输出的文件
- Plugin： 插件，在webpack构建流程中的特定时机会广播对应的事件，插件可以监听这些事件的发生，在特定的时机做对应的事情

### 2.entry
入口：webpack会根据入口，递归解析所有入口的依赖模块
通过AST解析，遇到引入的文件，然后再递归解析
- 打包后的文件是立即执行函数，参数是入口文件的列表
- webpack实现了自己的一套module，export。在解析文件的时候会包裹进去，兼容CommonJS和ES Module规范
- 递归解析的过程中，会将解析过的模块加入缓存，避免多次解析

### 3.loader
loader：根据规则对模块进行转换输出
- 规则执行顺序默认从右往左，从后往前
- 条件匹配：通过test，include，exclude，匹配规则
- 一个loader只负责一种转换，上一个loader会把转换的结果传递给下一个loader
![[loader.png]]
### 4.Plugin
插件：实现大部分功能
- 通过Tapable在webpack的流程中间注册事件，等webpack执行到此阶段时，广播通知注册了事件的插件。
- 插件可以修改，增删文件

### 5.webpack打包过程
![[webpack打包过程.png]]



## 二.webpack的构建流程

1.  解析参数，Webpack CLI 启动打包流程；
2.  载入 Webpack 核心模块，创建 Compiler 对象；
3.  使用 Compiler 对象开始编译整个项目；
4.  从入口文件开始，解析模块依赖，形成依赖关系树；
5.  递归依赖树，将每个模块交给对应的 Loader 处理；
6.  合并 Loader 处理完的结果，将打包结果输出到 dist 目录。

![[webpack.jpg]]




## 三.webpack的打包优化
### 优化打包
#### 1,缩小文件的搜索范围
- 引入文件后缀填写完整
- loader匹配规则，缩小范围
#### 2.忽略对没有采用模块化的包
noParse
#### 3.DllPlugin：使用动态链接
- 一些不常修改的文件使用DllPlugin打包出单独的文件
- 这样如果文件无修改，就可以不用再次打包
- 一般单独使用一个配置文件，webpack_dll.conf.js
#### 5. cache-loader缓存一些性能开销较大的 loader ，或者是使用 hard-source-webpack-plugin为模块提供一些中间缓存

#### 6.多进程打包
	**配置HappyPack将Loader单进程转换为多进程**，好处是`释放CPU多核并发的优势`。在使用`webpack`构建项目时会有大量文件需解析和处理，构建过程是计算密集型的操作，随着文件增多会使构建过程变得越慢
#### 7.可视化分析**BundleAnalyzer**
	`找出导致体积过大的原因`。从而通过分析原因得出优化方案减少构建时间

### 生产环境
#### 1.代码压缩
#### 2.Tree Shaking:去除多余代码
**Tree-shaking 实现的前提是 ES Modules**
``` js
// ./webpack.config.js
   module.exports = {
    
	 // ... 其他配置项
		  optimization: {

	// 模块只导出被使用的成员
		 usedExports: true,
		// 尽可能合并每一个模块到一个函数中
		concatenateModules: true,
		// 压缩输出结果
		 minimize: false
		}
    }
```

#### 3.css，图片等静态文件单独打包输出

#### 4.将node_module依赖包和业务代码分开打包

#### 5 .路由分块，代码懒加载
import() 实现动态导入

#### 6.提取公共代码，打包

#### 7.sideEffects
**Tree-shaking 只能移除没有用到的代码成员，而想要完整移除没有用到的模块，那就需要开启 sideEffects 特性了**

- 代码只是起导出模块的作用，没有其他逻辑执行

#### 8.根据需要引入polyfill
`@babel/preset-env`提供的`useBuiltIns`可按需导入`Polyfill`
-    **false**：无视`target.browsers`将所有`Polyfill`加载进来
-    **entry**：根据`target.browsers`将部分`Polyfill`加载进来(仅引入有浏览器不支持的`Polyfill`，需在入口文件`import "core-js/stable"`)
-    **usage**：根据`target.browsers`和检测代码里ES6的使用情况将部分`Polyfill`加载进来(无需在入口文件`import "core-js/stable"`)



## 四.webpack热更新原理
[热更新](https://zhuanlan.zhihu.com/p/30669007)
- webpack监听到文件修改，根据配置重新打包，将打包好的代码通过JS对象形式存储在内存中
- webpack-dev-middleware 调用 webpack 暴露的 API对代码变化进行监控，并且告诉 webpack，将代码打包到内存中。
- webpack-dev-server在浏览器端和服务端建立了一个websocket长连接，将 webpack 编译打包的各个阶段的状态信息告知浏览器端。务端传递的最主要信息还是新模块的 hash 值
- 热更新插件接收到hash值后，向服务端发送请求，获得包含热更新模块的hash值。然后再通过JSONP的请求拿到热更新的代码
- 进行比较新旧的hash值，然后决定是否更新。如果要更新，则需要将就模块的依赖跟新到新模块。**代码是以对象的形式存在内存中，所以只需要将依赖旧对象的依赖关系指向新模块对戏就可以**
- 当 HMR 失败后，回退到 live reload 操作，也就是进行浏览器刷新来获取最新打包代码。


![[热更新.jpg]]

## 五.webpack5的新特性
https://juejin.cn/post/6924258563862822919#heading-17
### 1..asset module
资源模块（asset module）是一种模块类型，它允许使用资源文件（字体、图标等）而无需配置额外 loader
-   raw-loader：允许将文件处理成一个字符串导入
-   file-loader：将文件打包导到输出目录，并在 import 的时候返回一个文件的 URI
-   url-loader：当文件大小达到一定要求的时候，可以将其处理成 base64 的 URIS ，内置 file-loader
### 2.文件缓存
内置 FileSystem Cache 能力加速二次构建
### 3.内置 WebAssembly 编译及异步加载能力（sync/async）
WebAssembly被设计为一种面向 web 的二进制的格式文件，以其更接近于机器码而拥有着更小的文件体积和更快速的执行效率
### 4.移除了 Node.js Polyfills
webpack4中有很多node端的Polyfill，增加了体积
### 5.资源打包策略更优，构建产物更“轻量”
### 6.深度 Tree Shaking 能力支持
Webpack5 能够支持深层嵌套的 export 的 Tree Shaking
#### 7.更友好的 Long Term Cache 支持性，chunkid 不变
Webpack5 之前，文件打包后的名称是通过 ID 顺序排列的，一旦后续有一个文件进行了改动，那么必将造成后面的文件打包出来的文件名产生变化，即使文件内容没有产生改变。因此会造成资源的缓存失效。

Webpack5 有着更友好的长期缓存能力支持，其通过 hash 生成算法，为打包后的 modules 和 chunks 计算出一个短的数字 ID ，这样即使中间删除了某一个文件，也不会造成大量的文件缓存失效
### 8.支持 Top Level Await，从此告别 async
``` js
// 开启 top level await 之前 
import i18n from 'XXX/i18n-utils' 
(async () => { 
	// 国际化文案异步初始化逻辑 await i18n.init({/* ... */}) 
		root.render(<AppContainer />) 
})()
	// 开启 top level await 之后 
import i18n from 'XXX/i18n-utils' 
await i18n.init({/* ... */}) 
root.render(<AppContainer />)

```

### 9.模块联邦
多个独立的构建可以组成一个应用程序，这些独立的构建之间不应该存在依赖关系，因此可以单独开发和部署它们
如果复用相同的模块，则可以不用每个都打包如下图
https://juejin.cn/post/6919651702303883272

![[webpack-联邦模式.jpg]]



https://juejin.cn/post/6981673766178783262#heading-3