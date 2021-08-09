# Promise
## 一.实现一个Promise
[参考](https://zhuanlan.zhihu.com/p/144058361)
- 三种状态， padding，resolved， rejected，一旦改变，不可逆
- Promise.then返回一个新promise，且传递函数的值给到下个promise

``` js
const PADDING = 'padding'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

function Promise1(executor) {
	this.status = PADDING // 初始状态
	this.data = undefined // 初始数据
	this.resolvedCb = [] // resolved的回调
	this.rejectedCb = [] // rejected的回调
	let self = this

	function resolve(value) {
		if (self.status === PADDING) {
			self.status = FULFILLED
			self.data = value
			self.resolvedCb.forEach(cb => {
				cb(value)
			})
		}
	}

	function reject(reason) {
		if (self.status === PADDING) {
			self.status = REJECTED
			self.data = reason
			self.rejectedCb.forEach(cb => {
				cb(reason)
			})
		}
	}

	// 执行
	try {
		executor(resolve, reject)
	} catch (error) {
		reject(error)
	}
}

// then: 方法返回一个Promise对象
Promise1.prototype.then = function (resolved, rejected) {
	let self = this
	resolved = typeof resolved === 'function' ? resolved : function () {}
	rejected = typeof rejected === 'function' ? rejected : function () {}
	let promise2 = new Promise1((resolve, reject) => {
		if (self.status === FULFILLED) {
			setTimeout(() => {
				try {
					let x = resolved(self.data)
					resolvedPromise(promise2, x, resolve, reject)
				} catch (error) {
					reject(x)
				}
			})
		}
		if (self.status === REJECTED) {
			setTimeout(() => {
				try {
					let x = rejected(self.data)
					resolvedPromise(promise2, x, resolve, reject)
				} catch (error) {
					reject(x)
				}
			})
		}
		// padding状态，收级回调函数
		if (self.status === PADDING)  {
			self.resolvedCb.push(() => {
				if (self.status === FULFILLED) {
					setTimeout(() => {
						try {
							let x = resolved(self.data)
							resolvedPromise(promise2, x, resolve, reject)
						} catch (error) {
							reject(x)
						}
					})
				}
			})
			self.rejectedCb.push(() => {
				setTimeout(() => {
					try {
						let x = rejected(self.data)
						resolvedPromise(promise2, x, resolve, reject)
					} catch (error) {
						reject(x)
					}
				})
			})
		}
	})
	return promise2
}



// 处理promise的步骤
// 1.promise === x 循环引用
// 2.x 为普通类型，那么直接传递
// 3.x为promise，那么会在执行promise
function resolvedPromise(promise, x, resolve, reject) {
	if(promise === x) return reject(new Error('循环引用了'))
	if (x && typeof x === 'object' || typeof x === 'function') {
			// 引用类型
			try {
				let then = x.then;
				// 定义标志，防止重读调用
				let used = false
					if(typeof then === 'function') { // 说明是Promise,那么就执行promise
						then.call(x, (y) => {
							if(used) return;
							used = true
							// y 也有可能是promise
							resolvedPromise(promise, y, resolve, reject)
						},(e) => {
							if(used) return;
							used = true
							reject(e)
						})
					} else {
						if(used) return;
						used = true
						resolve(x)
					}
				} catch (error) {
				if(used) return;
				used = true
				reject(error)
			}
		} else { // 普通类型
			resolve(x)
	}
}
```


## 二.在Promise的基础上实现主要API
### Promise.all
``` js
// 全部成功则进入resolved，一个失败则进入rejected

Promise.prototype.all = function(array) {
	if(!array.length) return Promise.reject('array is empty')
	let len = array.length
	let index = 0, resArr = []
	for(let i = 0; i < len;i++) {
		let p = array[i]
		p.then((res) => {
			if(index < 0) return
			resArr.push(res)
			index++
			if(index >= len) {
				return Promise.resolve(...resArr)
			}
		}).catch((error) => {
			if(index < 0) return
			index = -1
			return Promise.reject(error)
		})
	}
}
```
### Promise.race
``` js
// 有一个状态改变则进入对于的状态，其他的不管

Promise.prototype.race = function(array) {
	if(!array.length) return Promise.reject('array is empty')
	let len = array.length
	let index = 0,
	for(let i = 0; i < len;i++) {
		let p = array[i]
		p.then((res) => {
			if(index > 0) return
			index++
			return Promise.resolve(res)
		}).catch((error) => {
			if(index > 0) return
			index++
			return Promise.reject(error)
		})
	}
}
```
### Promise.any
``` js
// 有一个成功则成功，全部失败则会进入失败

Promise.prototype.any = function(array) {
	if(!array.length) return Promise.reject('array is empty')
	let len = array.length
	let index = 0,
	for(let i = 0; i < len;i++) {
		let p = array[i]
		p.then((res) => {
			if(index < 0) return;
			index = -1
			return Promise.resolve(res)
		}).catch((error) => {
			if(index < 0) return;
			index++
			if(index >= len) {
				return Promise.reject(error)
			}
		})
	}
}
```
