---
title: jQueryMobile笔记
tags: [jquery]
date: 2016/04/03
---

### page

//页面打开后展现形式
data-rel="dialog"
//开始预加载
data-prefetch="true"
//页面跳转时动画样式，更多样式请访问官方站点
data-transition="slidefade"
//如果在视图中设置以下两个属性，则会默认在header与footer的左侧显示后退按钮
data-add-back-btn="true"
data-back-btn-text="后退"

//示例

```
<a href="second_page.html" data-rel="dialog" data-prefetch="true" data-transition="slidefade">切换到第二个视图</a>
```

### button

//定义元素为按钮
data-role="button"
//按钮上的图标
data-icon="arrow-l"
//图标定位
data-iconpost="left/right/top/bottom/notext"
//内联样式
data-inline="true"
//
data-role="back"

/*自定义icon*/

```
.ui-icon-icon英文名称{
    background-image:url(18*18.png) no-repeat;
}
```

//定义按钮组
data-role="controlgroup"

```
<div data-role="controlgroup">
  <a href="#" data-role="button">确定</a>
  <a href="#" data-role="button">取消</a>
  <a href="#" data-role="button">返回</a>
</div>
```

//水平排列按钮组
data-type="horizontal"

```
<div data-role="controlgroup" data-type="horizontal">
  <a href="#" data-role="button">确定</a>
  <a href="#" data-role="button">取消</a>
  <a href="#" data-role="button">返回</a>
</div>
```

//元素浮动
data-position="fixed"

//全屏
data-fullscreen="true"

//布局
.ui-grid-a/b/c/d
.ui-block-a/b/c/d/e

//列子，三行五列

```
<div class="ui-grid-d">
	<section class="ui-block-a">
		<a href="#" data-role="button" data-theme="a">1</a>
	</section>
	<section class="ui-block-b">
		<a href="#" data-role="button" data-theme="b">2</a>
	</section>
	<section class="ui-block-c">
		<a href="#" data-role="button" data-theme="a">3</a>
	</section>
	<section class="ui-block-d">
		<a href="#" data-role="button" data-theme="b">4</a>
	</section>
	<section class="ui-block-e">
		<a href="#" data-role="button" data-theme="a">5</a>
	</section>
	<section class="ui-block-a">
		<a href="#" data-role="button" data-theme="b">6</a>
	</section>
	<section class="ui-block-b">
		<a href="#" data-role="button" data-theme="a">7</a>
	</section>
	<section class="ui-block-c">
		<a href="#" data-role="button" data-theme="b">8</a>
	</section>
	<section class="ui-block-d">
		<a href="#" data-role="button" data-theme="a">9</a>
	</section>
	<section class="ui-block-e">
		<a href="#" data-role="button" data-theme="b">10</a>
	</section>
	<section class="ui-block-a">
		<a href="#" data-role="button" data-theme="a">11</a>
	</section>
	<section class="ui-block-b">
		<a href="#" data-role="button" data-theme="b">12</a>
	</section>
	<section class="ui-block-c">
		<a href="#" data-role="button" data-theme="a">13</a>
	</section>
	<section class="ui-block-d">
		<a href="#" data-role="button" data-theme="b">14</a>
	</section>
	<section class="ui-block-e">
		<a href="#" data-role="button" data-theme="a">15</a>
	</section>
</div>
```

//例子 九宫格

```
<div class="ui-grid-b">
	<section class="ui-block-a">
		<a href="#" data-role="button" data-theme="a" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-b">
		<a href="#" data-role="button" data-theme="b" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-c">
		<a href="#" data-role="button" data-theme="c" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-a">
		<a href="#" data-role="button" data-theme="a" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-b">
		<a href="#" data-role="button" data-theme="b" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-c">
		<a href="#" data-role="button" data-theme="c" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-a">
		<a href="#" data-role="button" data-theme="a" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-b">
		<a href="#" data-role="button" data-theme="b" data-icon="home" data-iconpos="top">home</a>
	</section>
	<section class="ui-block-c">
		<a href="#" data-role="button" data-theme="c" data-icon="home" data-iconpos="top">home</a>
	</section>
</div>
```

//折叠块
data-role="collapsible"

//例子

```
<div data-role="collapsible">
	<h3>折叠</h3>
	<p>内容</p>
</div>
```

//手风琴
data-role="collapsible-set"
        data-role="collapsible"

//例子

```
<div data-role="collapsible-set">
	<section data-role="collapsible">
		<h3>手风琴</h3>
		<p>内容1</p>
	</section>
	<section data-role="collapsible">
		<h3>手风琴</h3>
		<p>内容2</p>
	</section>
	<section data-role="collapsible">
		<h3>手风琴</h3>
		<p>内容3</p>
	</section>
</div>
```

//开关按钮
data-role="fieldcontain"

//例子

```
<div data-role="fieldcontain">
	<label>toggle switches:</label>
	<select data-role="slider">
		<option value="off">off</option>
		<option value="on">on</option>
	</select>
</div>
```

查看demo请用手机扫描下方二维
![img](http://www.limeng.pw/wp-content/uploads/2014/12/generate.gif)
