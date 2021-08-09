# Koa-compose
## 作用： 将中间件组合起来，从而实现洋葱模型
``` js
function compose(middleware) {
	return function(ctx, next) {
		let index = -1
		function dispatch(i) {
			if(index > i) return Promise.reject(new Error('一个中间建中多次调用next'))
			index = i
			let fn = middleware[i]
			if(i === middleware.length) fn = next
			if(!fn) return Promise.reject()
			try {
				return Promise.resolve(fn(ctx, dispatch.bind(null, i + 1)))
			} catch (error) {
			return Promise.reject(error)
			}
		}
		return dispatch(0)
	}
}
```