# BFC，IFC
## 一.BFC 块级上下文
### 1.BFC定义
- 是页面中的一块渲染区域，有自己的一套布局规则
- **BFC是一个独立的布局环境，BFC内部的元素布局与外部互不影响**

### 2.BFC的布局规则
- 内部的box会在垂直方向一个接着一个的放置
- 盒子垂直方向的距离由margin决定，属于同一个BFC的相邻的盒子的margin会发生重叠
- BFC的区域不和浮动盒子重叠
- BFC是页面上一个独立的容器，里面和布局和容器外的布局互不干扰
- 计算BFC元素高度时，浮动子元素也会参与计算
- 每个盒子的左边界都会紧挨着BFC盒子的左边界，浮动元素也是

### 3.BFC产生的条件
- 根元素
- 浮动的元素：float：left，float：right
- display: inline-block
- position的值不是static或者relative
- overflow不为visible的元素
- 弹性布局：flex。网格布局：grid的子元素
### 4.作用
- 清除浮动
- 避免margin的塌陷


## 二.IFC 内联格式化上下文
### 1.IFC的形成条件
块级元素中仅包含内联级别元素

### 2.渲染规则
- 子元素在水平方向上一个接一个排列，在垂直方向上，自顶向下排列
- 节点无法声明宽高，margin只在水平方向有效
- 能把在一行上的框都完全包含进去的一个矩形区域，被称为该行的线盒（line box）。线盒的宽度是由包含块（containing box）和与其中的浮动来决定
-   IFC 中的 line box 一般左右边贴紧其包含块，但 float 元素会优先排列。
-   IFC 中的 line box 高度由 line-height 计算规则来确定，同个 IFC 下的多个 line box 高度可能会不同；
-   当内联级盒子的总宽度少于包含它们的 line box 时，其水平渲染规则由 text-align 属性值来决定
-   当一个内联盒子超过父元素的宽度时，它会被分割成多盒子，这些盒子分布在多个 line box 中。如果子元素未设置强制换行的情况下，inline box 将不可被分割，将会溢出父元素
### 3.应用场景
-   水平居中：当一个块要在环境中水平居中时，设置其为 inline-block 则会在外层产生 IFC，通过 text-align 则可以使其水平居中。
-   垂直居中：创建一个 IFC，用其中一个元素撑开父元素的高度，然后设置其 vertical-align: middle，其他行内元素则可以在此父元素下垂直居中。

## 三.清除浮动
### 1.让父元素形成BFC环境
- overflow： hiden
- position定位
- display：flex等
- 给父元素设置高
### 2.clear: both
- clear: both 会让非浮动元素在浮动元素下面
``` css
.block {
	zoom: 1;
}
.block::after {
	content: '';
	display: table;
	clear: both;
}
```
## 四.消除margin的塌陷
### 1.让父元素形成BFC环境
### 2.父元素加上border