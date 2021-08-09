# new、apply、call、bind
![[call,apply,bind.png]]
## 1.new
### 特点
- 创建一个新对象
- 将this指向这个对象
- 执行函数
- 返回对象
### 实现
``` js
function _new(fn, ...args) {
	 if(typeof fn !== 'function'){
	 	return new Error('fn must be a funtion')
	 }
	 // 创造对象，原型指向
	 let obj = new Object()
	 obj.__proto__ = Object.create(fn.prototype)
	 // 执行函数
	 let res = fn.call(obj, ...args)
	 // 返回
	 return typeof res === 'object' || typeof res === 'function' ? res : obj
}
```

## 2.call apply
### 特点
- 都将改变函数的this，且立即执行
- apply接收的参数是数组，call则是多参
### 实现
``` js
Function.prototype.call = function (context, ...args) {
	// 上下文
	context = context || window
	context.fn = this
	// 执行
	let res = context.fn(...args)
	delete context.fn
	return res
}

Function.prototype.apply = function (context, args) {
	// 上下文
	context = context || window
	context.fn = this
	// 执行
	let res = context.fn(args)
	delete context.fn
	return res
}
```

## 3.bind的实现
### 特点
- 改变执行环境的this
- 返回一个函数，延迟执行
### 实现
``` js

Function.prototype.bind = function (context, ...args) {
	// 上下文
	context = context || window
	context.fn = this
	// 执行
	let res = function () {
		let res1 = context.fn(...args)
		delete context.fn
		return res1
	}
	// 返回函数
	return res
}
```

