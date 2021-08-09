# XSS、CSRF
## 一.XSS: 跨站脚本攻击
### 1.XSS的简介
通过加载非正常的JS脚本，对网站进行攻击。
- 获取页面数据：dom,获取dom的内容
- 获取用户的Cookies：document.cookie
- 劫持前端逻辑：改变操作等
- 发送请求：图片,form等

#### 危害
- 偷取网站的任意数据
- 偷取用户的信息，cookie
- 欺骗用户

#### 分类
- 反射型：
	- 非持久
	- 通过url注入脚本，当网站获取url参数并在页面上未转码直接展示时，就会被攻击
	- 攻击方式：通过url注入，可能时短链接
- 存储型
	- 持久型
	- 攻击方式：通过数据存储到数据库，读取的时候就可能会执行

### 2.XSS注入的方式
#### html的节点内容
 动态生成,包含用户输入信息
 
		<div>
		123 <script>alert(123)</script>
		</div>
#### html 属性
节点的属性,由用户输入的数据组成

	    <img src="xxx.jpg">
        输入: 1" onerror="alert1
        从而属性提前闭合
        <img src="1" onerror="alert1">
		
#### js代码
后台数据返回，导致语句提前闭合
	
	    var data = "123"
        
        数据:123";alert(1)"
            提前闭合
        
        var data = "123";alert(1)""
		
#### 富文本
保留html
html 就有 xss 攻击的风险


### 3.XSS的防御
#### 禁止脚本操作Cookie
cookie：HttpOnly
#### X-XSS-Protection
-  HTTP **X-XSS-Protection** 响应头 是浏览器的特性
- 浏览器自带拦截，
- 0(关闭), 1(开启), 1;mode=block（检测到违规，则停止加载页面）

#### 转义
对HTML或者HTML的属性进行转义处理
- 转义<>

        &lt; 和 &gt;
        时机: 数据存入时,数据显示时
	
- 转义 单双引号，空格
   
		 引号 " ' 
			&quto; &#39;
			空格
			&#32;
			&(放到最前面)
			&amp;
			
#### 设置富文本的白名单，黑名单
- html节点白名单
- 属性白名单
- 黑名单，或者非白名单的数据进行过滤

		一般在输入的时候过滤,
		工具库:cheerio, xss
			
#### CSP：内容安全策略
- 设置安全的资源范围，图片,js,css，媒体资源等
- 参考：
	- https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP
	- https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy

## 二.CSRF: 跨域伪装请求
### 1.简介
   跨站请求伪造
    (其他网站在用户不知情时对目标网站发出一些请求)
### 2.原理
1.用户登录A网站
2.A网站确认身份
3.B网站页面想A网站发起请求
	(带A网站的身份- cookie)
	A网站表单, img a 之类的都可以
	
### 特点
- B网站向A网站请求
- 带有A网站的cookies
- 不访问A网站的前端
- referer为B网站
	
### 危害
- 利用用户的登录态
- 用户不知情
- 完成业务请求
- 盗取用户的资金
- 冒充用户发帖背锅
- 损害网站名誉
### 防御
- 禁止第三方网站带cookies：cookie的 same-site 属性
- 在前端页面加入验证信息，（B网站无法获取的）
	- 重要操作加上验证码
	- token
- 验证referer： 可以修改，不一定准

## 3.Cookie
- xss 可以操作脚本，所以要设置HttpOnly
- CSRF：在B网站通过表单，a,, img等发A网站的请求。所以要设置不允许其他网站带自己网站的cookie，cookie的 same-site 属性
