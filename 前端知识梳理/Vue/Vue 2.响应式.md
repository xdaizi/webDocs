# Vue2.响应式

## 流程
### 1.初始化
- 初始化data数据 observe
- 变换成响应式对象 defineReactive
- 核心：Object.defindProperty(obj, key, {get, set})
- get : 收集依赖于响应式对象的watcher
- set： 触发相应watcher的更新
- dep: 管理依赖（watcher）
- watcher： 对应的更新（改变视图）

### 2.变化
- 触发set
- 更新watcher的update方法





## 手写代码
``` js
function Vue(opt) {
	let vm = this;
	let subId = 0, watcherId = 0
	init(opt, vm)
	// 初始化
	function init(opt, vm) {
		let {el, data, render} = opt
		vm.el = document.querySelector(el)
		render = render.bind(vm)
		vm._data = data()
		// 转化为响应式对象
		observe(vm._data)
		let dom = render(createEle.bind(vm))
		vm.el.appendChild(dom)
	}
	function createEle(vnode) {
		let wrapper = document.createDocumentFragment()
		for(let i = 0, l = vnode.length; i < l; i++) {
			let node = vnode[i]
			let dom = document.createElement(node.tag)
			dom.className = node.className
			new Watcher(this, () => {
				dom.innerText = this._data[node.text]
			})
			wrapper.appendChild(dom)
		}
		return wrapper
	}
	function observe(data) {
		return new Obverser(data)
	}
	// 循环遍历转化
	function Obverser(data) {
		for(let key in data) {
			defineReactive(data, key)
		}
	}
	// 劫持
	function defineReactive(obj, key) {
		let dep = new Dep();
		let val = obj[key]
		Object.defineProperty(obj, key, {
			enumerable: true,
			configurable: true,
			get() {
				console.log('get')
				// 收集依赖
				dep.depend()
				return val
			},
			set(newVal) {
				if(newVal === val) return;
				val = newVal;
				// 触发
				dep.notify()
			}
		})
	}
	function Dep() {
		this.subId = subId++
		this.subs = [];
		// 收集依赖于响应式遍历的对象
		this.depend = function() {
			// Dep.target = 当前的Watcher
			if(Dep.target) {
				Dep.target.addDep(this)
			}
		}
		this.addSub = function(watcher) {
			this.subs.push(watcher)
		}
		// 响应
		this.notify = function() {
		for(let i = 0, l = this.subs.length; i < l; i++) {
			let watcher = this.subs[i]
				watcher.update()
			}
		}
	}
	
	function Watcher(vm, update) {
		this.watcherId = watcherId++
		this.vm = vm
		// 设置当前的watcher
		Dep.target = this;
		// 添加
		this.addDep = function(dep) {
			dep.addSub(this)
		}
		// 更新
		this.update = function() {
			console.log('Watcher 更新！！！')
			update()
		}
		this.value = update()
		Dep.target = null
	}
 }

let vue = new Vue({
	el:'#app',
	data() {
		return {
			text: 123
		}
	},
	render (h) {
		let app = [
			{
				tag: 'div',
				className: 'box',
				text: 'text',
			}
		]
		return h(app)
	}
})

setTimeout(() => {
	vue._data.text = 222
}, 2000)
```