---
title: 12个CSS高级技巧汇总
tags: [css]
date: 2015/01/24
---

**1、使用:not()在菜单上应用/取消应用边框**

先给每一个菜单项添加边框

```css
/* add border */
.nav li {
	border-right: 1px solid #666;
}
```

……然后再除去最后一个元素……

```css
//* remove border */
.nav li:last-child {
	border-right: none;
}
```

……可以直接使用:not()伪类来应用元素：

```css
.nav li:not(:last-child) {
	border-right: 1px solid #666;
}
```

这样代码就干净，易读，易于理解了。

当然，如果你的新元素有兄弟元素的话，也可以使用通用的兄弟选择符（~）：

```css
.nav li:first-child ~ li {
	border-left: 1px solid #666;
}
```

**2、给body添加行高**

你不需要分别添加 line-height 到每个```<p>```，```<h*>```等。只要添加到body即可：

```css
body {
	line-height: 1;
}
```

这样文本元素就可以很容易地从body继承。

**3、所有一切都垂直居中**

要将所有元素垂直居中，太简单了：

```css
html, body {
	height: 100%;
	margin: 0;
}

body {
	-webkit-align-items: center;
	-ms-flex-align: center;
	align-items: center;
	display: -webkit-flex;
	display: flex;
}
```

看，是不是很简单。

> 注：在IE11中要小心flexbox。

**4、逗号分隔的列表**

让HTML列表项看上去像一个真正的，用逗号分隔的列表：

```css
ul > li:not(:last-child)::after {
	content: ",";
}
```

对最后一个列表项使用:not()伪类。

**5、使用负的nth-child选择项目**

在CSS中使用负的 nth-child 选择项目1到项目n。

```css
li {
	display: none;
}

/* select items 1 through 3 and display them */
li:nth-child(-n+3) {
	display: block;
}
```

就是这么容易。

**6、对图标使用SVG**

我们没有理由不对图标使用SVG：

```css
.logo {
	background: url("logo.svg");
}
```

SVG对所有的分辨率类型都具有良好的扩展性，并支持所有浏览器都回归到IE9。这样可以避开.png、.jpg或.gif文件了。

**7、优化显示文本**

有时，字体并不能在所有设备上都达到最佳的显示，所以可以让设备浏览器来帮助你：

```css
html {
	-moz-osx-font-smoothing: grayscale;
	-webkit-font-smoothing: antialiased;
	text-rendering: optimizeLegibility;
}
```

注：请负责任地使用optimizeLegibility。此外，IE/Edge没有text-rendering支持。

**8、对纯CSS滑块使用max-height**

使用max-height和溢出隐藏来实现只有CSS的滑块：

```css
.slider ul {
	max-height: 0;
	overlow: hidden;
}

.slider:hover ul {
	max-height: 1000px;
	transition: .3s ease;
}
```

**9、继承box-sizing**

让box-sizing继承html：

```css
html {
	box-sizing: border-box;
}

*, *:before, *:after {
	box-sizing: inherit;
}
```

这样在插件或杠杆其他行为的其他组件中就能更容易地改变box-sizing了。

**10、表格单元格等宽**

表格工作起来很麻烦，所以务必尽量使用table-layout: fixed来保持单元格的等宽：

```css
.calendar {
	table-layout: fixed;
}
```

**11、用Flexbox摆脱外边距的各种hack**

当需要用到列分隔符时，通过flexbox的space-between属性，你就可以摆脱nth-，first-，和last-child的hack了：

```css
.list {
	display: flex;
	justify-content: space-between;
}

.list .person {
	flex-basis: 23%;
}
```

现在，列表分隔符就会在均匀间隔的位置出现。

**12、使用属性选择器用于空链接**

当`<a>`元素没有文本值，但href属性有链接的时候显示链接：

```css
a[href^="http"]:empty::before {
	content: attr(href);
}
```

相当方便。

> 些高级技巧在Chrome、Firefox、Safari、Edge的当前版本，以及IE11中都能有效工作。