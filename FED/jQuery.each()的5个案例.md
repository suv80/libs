---
title: jQuery.each()的5个案例
tags: [jquery]
date: 2016/04/04
---

本文将对 jQuery each() 函数作一个比较全面的介绍。 each() 函数是 jQuery 中最重要也是最常用的函数之一。通过本文你将明白为什么 each() 函数如此大放异彩，同时还将详细介绍如何使用 each() 函数。

### 什么是 jQuery .each()

jQuery 的 each() 函数用来遍历目标 jQuery 对象中的所有元素。在这里解释一下什么是 jQuery 对象，以防有读者还不太清楚。 jQuery 对象指的是包含一个或多个 DOM 元素的对象，并且暴露出所有的 jQuery 函数。 each() 函数非常适合操作多元素的 DOM 、任意数组的循环以及对象的属性。除了这个函数， jQuery 里还有一个同名的辅助函数，不需要事先选择或创建 DOM 元素就可以调用这个辅助函数。在接下来的部分将更详细地介绍它们。

### jQuery 的 .each() 语法

这里将结合实际例子讲解 .each() 的不同用法。

下面这个例子选中了网页上所有 div 标签，并打印每个 div 标签的 index 和 ID。输出的结果可能是： “div0:header”、 “div1:body”、 “div2:footer”。 jQuery 中 each() 函数的这种用法和效用函数的用法完全不同。

```
// DOM ELEMENTS
$('div').each(function (index, value) { 
  console.log('div' + index + ':' + $(this).attr('id')); 
});
```

下一个例子展示了效用函数的用法。在这个例子当中，把被循环的对象当作 each() 函数的第一个参数。这个例子展示了如何遍历一个数组：

```
// ARRAYS
var arr = [
   'one',
   'two',
   'three',
   'four',
   'five'
];
$.each(arr, function (index, value) {
  console.log(value);
 
  // Will stop running after "three"
  return (value !== 'three');
});
// Outputs: one two three
```

最后这个例子中，遍历了一个对象中的所有属性：

```
// OBJECTS
var obj = {
   one: 1,
   two: 2,
   three: 3,
   four: 4,
   five: 5
};
$.each(obj, function (index, value) {
  console.log(value);
});
// Outputs: 1 2 3 4 5
```

这一切都归结为提供了适当的回调函数。这个回调函数的上下文， this ，等于它的第二个参数，也就是当前的 value 值。通常上下文都是一个对象，所以得把原始值封装起来。也就是说，这个 value 值和上下文之间不存在严格相等关系。 回调函数的第一个参数是当前的 index，它可能是数组里一个数字或对象中一个字符串。

### 1.基本的 jQuery.each() 实例

一起来看看 each() 函数是如何处理一个 jQuery 对象的。第一个例子中选择了页面中所有的 a 标签，并打印出它们的 href 属性。

```
$('a').each(function (index, value){
  console.log($(this).attr('href'));
});
```

第二个例子中输出了网页上所有的外链 href 属性（这里我们假设只用了 HTTP 协议）：

```
$('a').each(function (index, value){
  var link = $(this).attr('href');
 
  if (link.indexOf('http://') === 0) {
        console.log(link);
  }
});
```

假设当前页面中有如下这些链接：

```
<a href="http://www.jquery4u.com">JQUERY4U</a>
<a href="http://www.phpscripts4u.com">PHP4U</a>
<a href="http://www.blogoola.com">BLOGOOLA</a>
```

那么第二个例子将输出如下结果：

```
http://jquery4u.com
http://www.phpscripts4u.com
http://www.blogoola.com
```

需要注意的是， 在 each() 当中使用 jQuery 对象的 DOM 元素时，必须对这些 DOM 元素再次封装。这是因为 jQuery 实际上只是将 DOM 元素封装成数组。使用 jQuery each() 方法其实就是像普通数组那样迭代这个数组。因此，无法在迭代器里得到封装好的元素。

### 2. jQuery.each() 数组实例

首先看看 each() 是如何处理一个普通数组的。

```
var numbers = [1, 2, 3, 4, 5, 6];
$.each(numbers , function (index, value){
  console.log(index + ':' + value); 
});
```

这段代码输出的结果是：0:1、 1:2、 2:3、 3:4、 4:5, 和 5:6。

这段代码没有特别的。数组带数字索引，所以我们从０开始向后取数字，一直取到 N – 1，其中 N 是这个数组中元素的个数。

### 3. jQuery.each() JSON 实例

有时可能遇到更复杂的数据结构，比如数组包含数组、对象包含对象、对象包含数组，或者数组包含对象。这些情况下， each() 方法是如何发挥作用的呢？

```
var json = [ 
 { 'red': '#f00' },
 { 'green': '#0f0' },
 { 'blue': '#00f' }
];
 
$.each(json, function () {
   $.each(this, function (name, value) {
      console.log(name + '=' + value);
   });
});
```

这个例子的输出结果是 red=#f00、 green=#0f0、 blue=#00f。

我们用嵌套调用 each() 方法，来处理这种嵌套结构。外层的 each() 调用是用来处理变量名为 JSON 的数组，内层的 each() 调用是用来处理嵌套在数组中的对象。本例当中，每个对象只有一个属性，但是一般来说有多个属性也是可以的。

### 4.jQuery.each() 类实例

这个例子展示了在下面 HTML 代码中，如何遍历所有 class 属性为 productDescription 的元素。

```
<div class="productDescription">Red</div>
<div>Pink</div>
<div class="productDescription">Orange</div>
<div class="generalDescription">Teal</div>
<div class="productDescription">Green</div>
```

我们使用 each() 辅助函数，取代在选择器中用 each() 方法。

```
$.each($('.productDescription'), function (index, value) { 
  console.log(index + ':' + $(value).text()); 
});
```

本例中的输出结果是0:Red, 1:Orange, 2:Green。

我们不一定非要带上 index 和 value。它们只是用来帮助判断当前正在迭代处理的 DOM 元素的参数。而且，这种情况下也能更方便的使用 each 方法。可以简写成这样：

```
$('.productDescription').each(function () { 
  console.log($(this).text());
});
```

在控制台上得到的结果如下：

```
Red
Orange
Green
```

同样地，必须把 DOM 元素封装到在一个新的 jQuery 实例中。使用 text() 方法将元素的文本输出。

### 5. jQuery .each() 延迟实例

在下面的这个例子中，当用户点击 ID 为 5demo 的元素时，所有的列表元素都会马上变成橙色。然后元素根据 index 值设置延迟时间 (0, 200, 400, … 毫秒) 逐个淡出。

```
$('#5demo').bind('click', function (e) {
  $('li').each(function (index) {
$(this).css('background-color', 'orange')
  .delay(index * 200)
  .fadeOut(1500);
  });
  e.preventDefault();
});
```

### 结论

编程时应该尽可能多地使用 each() 函数，这种高效的方法能为我们节省大量时间。除了 jQuery 之外，还可以使用 ECMAScript 5 数组中的 forEach() 方法。

> 谨记： $.each() 和 $(selector).each() 是用不同方式定义的两种不同的方法。

[http://web.jobbole.com](http://web.jobbole.com/85589/)