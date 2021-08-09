# WEB安全
## 1.常见攻击
- XSS
- CSRF
- 密码安全
	- https
	- 加盐，算法组合加密存储
		- 单MD5不安全，彩虹表对比
	- 密文存储：存储加密后的密码
	- 密码传输对比：密文传输
		- 对比加密后的密码
		- 前端预加密，防止明文传输，与后端存储加密最开始的一样
- 点击劫持： iframe
	- 特点：
		- 攻击网站用iframe嵌入别的网站.且iframe透明度
    为0,用户操作的是看到的网站,但其实是iframe嵌入的目标网站
		-    用户亲手操作， 用户不知情
	-  防御：
		-  验证码
		-  X-FRAME-OPTIONS 禁止内嵌
			
				 X-Frame-Options
				'DENY' --- 禁止内嵌
				'SAME-ORIGIN' -- 同一网站可以访问
				'SAMEORIGIN' --- 同一域允许内嵌
				'ALLOW-FROM 网站名' --- 允许某网站可以内嵌
		-  js判断禁止内嵌  
	
			        防止iframe引入
				top === window , top.location === window.location
				没有内嵌时,top === window
				内嵌时, top !== window

				--------------------
				if(top.location != window.location) {
					top.location = window.location
				}
				------------
				缺点:iframe的sendbox 属性可以禁止js
		