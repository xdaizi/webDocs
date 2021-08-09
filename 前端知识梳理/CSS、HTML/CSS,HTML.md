# CSS/HTML
## 一.html5新增特性
- audio，video 媒体标签
- 原生拖拽
	-  dragstart
	-  drag
	-  dragend
- 地理位置的获取
	-   latitude：纬度
	-   longitude：经度
	-   accuracy：经、纬度坐标的精度，以米为单位
- canvas的支持
- header，section，nav等语义化标签的支持
- 表单的类型的增加
	- type：**number**、**url**、**email**、**range**、**color**、**date**
	- 属性：**placeholder**、**required**、**pattern**、**min**、**max**、**height**、**width**
- DOM扩展
	- 自定义属性，data-
- classList
- 本地存储
	- loaclStroge
	- sessionStroge
## 二.CSS3新特性
[参考](https://juejin.cn/post/6844903518520901639#heading-4)
- 选择器
- 过渡，动画，变形
- 阴影，边框
- 媒体查询
- 布局等
## 三.Canvas 与 SVG 的比较
### 1.SVG
- SVG 是一种使用 XML 描述 2D 图形的语言。
- SVG 基于 XML，这意味着 SVG DOM 中的每个元素都是可用的。您可以为某个元素附加 JavaScript 事件处理器。
- 在 SVG 中，每个被绘制的图形均被视为对象。如果 SVG 对象的属性发生变化，那么浏览器能够自动重现图形。
### 2.Canvas
- Canvas 通过 JavaScript 来绘制 2D 图形。
- Canvas 是逐像素进行渲染的。
- 在 canvas 中，一旦图形被绘制完成，它就不会继续得到浏览器的关注。如果其位置发生变化，那么整个场景也需要重新绘制，包括任何或许已被图形覆盖的对象。
### 3.区别
#### Canvas
-   依赖分辨率
-   不支持事件处理器
-   弱的文本渲染能力
-   能够以 .png 或 .jpg 格式保存结果图像
-   最适合图像密集型的游戏，其中的许多对象会被频繁重绘
#### SVG
-   不依赖分辨率
-   支持事件处理器
-   最适合带有大型渲染区域的应用程序（比如谷歌地图）
-   复杂度高会减慢渲染速度（任何过度使用 DOM 的应用都不快）
-   不适合游戏应用
## 五.HTML5浏览器兼容性解决方案
所有浏览器 ，对无法识别的元素会作为内联元素自动处理
### 1.将HTML5新标签转化成自定义标签
``` html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
  <title>为 HTML 添加新元素</title>
  <script>document.createElement("myElement")</script>
  <style>
  myElement{
    display: block;
    background-color: #ddd;
    padding: 50px;
    font-size: 30px;
  } 
  </style> 
</head>

<body>

<h1>我的第一个标题</h1>

<p>我的第一个段落。</p>

<myElement>我的第一个新元素</myElement>

</body>
</html>
```
### 2.将新标签设置成块级元素
``` css
eader, section, footer, aside, nav, main, article, figure { display: block; }
```
### 3.针对IE浏览器，做条件判断
``` html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>渲染 HTML5</title>
  <!--[if lt IE 9]>
  <script src="http://apps.bdimg.com/libs/html5shiv/3.7/html5shiv.min.js"></script>
  <![endif]-->
</head>

<body>

<h1>我的第一个HTML5页面</h1>

<article>
Hello，world！
</article>

</body>
</html>
```


## CSS汇总复习
CSS三大特性
- 继承性
- 层叠性
- 优先性
https://juejin.cn/post/6941206439624966152#heading-10