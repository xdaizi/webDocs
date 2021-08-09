# CommonJs,AMD

## 一.CommonJs规范
node.js的模块系统，就是参照CommonJS规范实现的
[CommonJs](http://javascript.ruanyifeng.com/nodejs/module.html#toc1)
-   所有代码都运行在模块作用域，不会污染全局作用域。
	-  #### `module`、`exports`和`require`实现
-   模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
-   模块加载的顺序，按照其在代码中出现的顺序。同步加载
-   加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数
-   就近原则：依赖就近

## 二.AMD规范
由于CommonJs 是同步加载的，在浏览器端容易阻塞加载，所以不适用。
- 异步加载模块
- 依赖前置: CMD依赖置顶