# babel
## 一.babel简介
### 1.定义
Babel是一个JS的编译器
### 2.核心
AST: 抽象语法树
[Babel AST 格式](https://github.com/babel/babel/blob/main/packages/babel-parser/ast/spec.md)
### 3.特点
-   **可插拔**（Pluggable），比如 Babel 需要有一套灵活的插件机制，召集第三方开发者力量，同时还需要方便接入各种工具；
-   **可调式**（Debuggable），比如 Babel 在编译过程中，要提供一套 Source Map，来帮助使用者在编译结果和编译前源码之间建立映射关系，方便调试；
-   **基于协定**（Compact），Compact 可以简单翻译为基于协定，主要是指实现灵活的配置方式
### 4.主要作用
-   语法转换，一般是高级语言特性的降级；
    
-   Polyfill（垫片/补丁）特性的实现和接入；
    
-   源码转换，比如 JSX 等。
### 5.工作流程
- 源码编译AST
- 对AST进行修改
- 将修改后的AST生成code

babel只是转译新标准引入的语法，比如ES6的箭头函数转译成ES5的函数；而**新标准引入的新的原生对象，部分原生对象新增的原型方法，新增的API等（如Proxy、Set等），这些babel是不会转译的。需要用户自行引入polyfill来解决**
[babel文章][https://www.jianshu.com/p/e9b94b2d52e2)


![[babel编译流程.png]]


## 二.常见babel的分类
### 1.@babel/core
[@babel/core](https://babeljs.io/docs/en/babel-core) **是 Babel 实现转换的核心**，它可以根据配置，进行源码的编译转换

@babel/core 的能力由更底层的 **@babel/parser**、**@babel/code-frame**、**@babel/generator**、**@babel/traverse、@babel/types**等包提供
### 2.@babel/cli
[@babel/cli](https://babeljs.io/docs/en/babel-cli) **是 Babel 提供的命令行**，它可以在终端中通过命令行方式运行，编译文件或目录

@babel/cli 负责获取配置内容，并最终依赖了 @babel/core 完成编译。
### 3.@babel/parser: 将源码编译得到AST
require("@babel/parser").parse()方法可以返回给我们一个针对源码编译得到的 AST
 [@babel/parser](https://babeljs.io/docs/en/babel-parser)
### 4.@babel/traverse：修改AST
@babel/travers，@babel/types 提供修改AST节点的能力
[@babel/types](https://babeljs.io/docs/en/babel-types)
[@babel/traverse](https://babeljs.io/docs/en/babel-traverse)

### 5.@babel/generator：生成代码
[@babel/generator](https://babeljs.io/docs/en/babel-generator) **对新的 AST 进行聚合并生成 JavaScript 代码**
### 6.@babel/preset-env
配置需要支持的目标环境（一般是浏览器范围或 Node.js 版本范围），利用 babel-polyfill完成补丁的接入

[@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
### 7.@babel/polyfill
**@babel/polyfill 其实就是 core-js 和 regenerator-runtime 两个包的结合**

**@babel/polyfill 目前已经计划废弃**，新的 Babel 生态（@babel/preset-env V7.4.0 版本）鼓励开发者直接在代码中引入 core-js 和 regenerator-runtime

que

[@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill)

### 8.**@babel/runtime**
有 Babel 编译所需的一些运行时 helpers 函数，**供业务代码引入模块化的 Babel helpers 函数**
**babel-runtime：功能类似babel-polyfill，一般用于library或plugin中，因为它不会污染全局作用域**
### 9.@babel/plugin-transform-runtime
**重复使用 Babel 注入的 helpers 函数**，达到**节省代码大小**的目的
[@babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime)
```js
var _interopRequireDefault = require("@babel/runtime/helpers/interopRequireDefault");
    
var _classCallCheck2 = _interopRequireDefault(require("@babel/runtime/helpers/classCallCheck"));
    
var Person = function Person() {
    (0, _classCallCheck2.default)(this, Person);
};
```
-   @babel/plugin-transform-runtime 需要和 @babel/runtime 配合使用；
-   @babel/plugin-transform-runtime 用于编译时，作为 devDependencies 使用；
-   @babel/plugin-transform-runtime 将业务代码编译，引用 @babel/runtime 提供的 helpers，达到缩减编译产出体积的目的；
-   @babel/runtime 用于运行时，作为 dependencies 使用。


@babel/plugin-transform-runtime 和 @babel/runtime 的作用
- 代码瘦身
- 避免全局污染

### 10.其他
- @babel/plugin是 Babel 插件集合。
- **@babel/plugin-syntax-* 是 Babel 的语法插件**
- **@babel/plugin-proposal-* 用于编译转换在提议阶段的语言特性**
- **@babel/plugin-transform-* 是 Babel 的转换插件**
- [@babel/template](https://babeljs.io/docs/en/babel-template) 封装了基于 AST 的模板能力，可以将字符串代码转换为 AST。比如在生成一些辅助代码（helper）时会用到这个库。
    
-   @[babel/node](https://babeljs.io/docs/en/babel-node) 类似 Node.js Cli，@babel/node 提供在命令行执行高级语法的环境，也就是说，相比于 Node.js Cli，它加入了对更多特性的支持。
    
-   [@babel/register](https://babeljs.io/docs/en/babel-register) 实际上是为 require 增加了一个 hook，使用之后，所有被 Node.js 引用的文件都会先被 Babel 转码。







