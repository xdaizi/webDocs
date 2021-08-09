# LRU缓存

## 1.原理
 LRUCache 类：
- LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存
- int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
- void put(int key, int value) 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

## 2.实现
### 利用数组和hash
``` js
/**

 * @param {number} capacity

 */

 var LRUCache = function(capacity) {
	this.size = capacity
	this.hash = new Map()
	this.array = []
};

  

/** 

 * @param {number} key

 * @return {number}

 */

LRUCache.prototype.get = function(key) {
	if(!this.hash.has(key)) return -1
	let val = this.hash.get(key)
	let index = this.array.indexOf(key)
	this.array.splice(index, 1)
	this.array.unshift(key)
	return val
};

  

/** 

 * @param {number} key 

 * @param {number} value

 * @return {void}

 */

LRUCache.prototype.put = function(key, value) {
	let oldVal = this.get(key)
	if(oldVal === -1) {
		if(this.array.length >= this.size) {
			let oldkey = this.array.pop()
			this.hash.delete(oldkey)
		}
		this.array.unshift(key)
		this.hash.set(key, value)
	} else {
		this.hash.set(key, value)
	}
};

  

/**

* Your LRUCache object will be instantiated and called as such:
 * var obj = new LRUCache(capacity)
 * var param_1 = obj.get(key)
 * obj.put(key,value)
 */

```

### 利用hash，时间复杂度O(n)
``` js
/**

 * @param {number} capacity

 */

 var LRUCache = function(capacity) {
	this.capacity = capacity;
	this.map = new Map();
};

  

/** 

 * @param {number} key

 * @return {number}

 */

LRUCache.prototype.get = function(key) {
	if(!this.map.has(key)) return - 1
	let val = this.map.get(key)
	this.map.delete(key)
	this.map.set(key, val)
	return val
};

  

/** 

 * @param {number} key 

 * @param {number} value

 * @return {void}

 */

LRUCache.prototype.put = function(key, value) {
	if(this.map.has(key)) {
		this.map.delete(key)
	}
	this.map.set(key, value)
	if(this.map.size > this.capacity) {
		let keys = this.map.keys()
		let oldKey = keys.next().value
		this.map.delete(oldKey)
	}
};

  

/**

 * Your LRUCache object will be instantiated and called as such:

 * var obj = new LRUCache(capacity)

 * var param_1 = obj.get(key)

 * obj.put(key,value)

 */
```