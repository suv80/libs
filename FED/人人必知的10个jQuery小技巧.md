---
title: 人人必知的10个jQuery小技巧
tags: [javascript,jquery]
date: 2015/02/07
---

收集的10个jQuery小技巧/代码片段，可以帮你快速开发。

### 1.返回顶部按钮

你可以利用animate和scrollTop来实现返回顶部的动画，而不需要使用其他插件。

```
// Back to top
$('a.top').click(function () {
  $(document.body).animate({scrollTop: 0}, 800);
  return false;
});

<!-- Create an anchor tag -->
<a class="top" href="#">Back to top</a>

```

改变scrollTop的值可以调整返回距离顶部的距离，而animate的第二个参数是执行返回动作需要的时间(单位：毫秒)。

### 2. 预加载图片

如果你的页面中使用了很多不可见的图片（如：hover显示），你可能需要预加载它们：

```
$.preloadImages = function () {
  for (var i = 0; i < arguments.length; i++) {
    $('<img>').attr('src', arguments[i]);
  }
};
```

```$.preloadImages('img/hover1.png', 'img/hover2.png');```

### 3. 检查图片是否加载完成

有时候你需要确保图片完成加载完成以便执行后面的操作：

```
$('img').load(function () {
  console.log('image load successful');
});
```

你可以把img替换为其他的ID或者class来检查指定图片是否加载完成。

### 4. 自动修改破损图像

如果你碰巧在你的网站上发现了破碎的图像链接，你可以用一个不易被替换的图像来代替它们。添加这个简单的代码可以节省很多麻烦：

```
$('img').on('error', function () {
  $(this).prop('src', 'img/broken.png');
});
```

即使你的网站没有破碎的图像链接，添加这段代码也没有任何害处。

### 5. 鼠标悬停(hover)切换class属性

假如当用户鼠标悬停在一个可点击的元素上时，你希望改变其效果，下面这段代码可以在其悬停在元素上时添加class属性，当用户鼠标离开时，则自动取消该class属性：

```
$('.btn').hover(function () {
  $(this).addClass('hover');
  }, function () {
    $(this).removeClass('hover');
  });
```

你只需要添加必要的CSS代码即可。如果你想要更简洁的代码，可以使用toggleClass 方法：

```
$('.btn').hover(function () { 
  $(this).toggleClass('hover'); 
});
```

注：直接使用CSS实现该效果可能是更好的解决方案，但你仍然有必要知道该方法。

### 6. 禁用 input 字段

有时你可能需要禁用表单的submit按钮或者某个input字段，直到用户执行了某些操作（例如，检查“已阅读条款”复选框）。可以添加disabled属性，直到你想启用它时：

```
$('input[type="submit"]').prop('disabled', true);
```

你要做的就是执行removeAttr方法，并把要移除的属性作为参数传入：

```
$('input[type="submit"]').removeAttr('disabled');
```

### 7. 阻止链接加载

有时你不希望链接到某个页面或者重新加载它，你可能希望它来做一些其他事情或者触发一些其他脚本，你可以这么做：

```
$('a.no-link').click(function (e) {
  e.preventDefault();
});
```

### 8. 切换fade/slide

fade和slide是我们在jQuery中经常使用的动画效果，它们可以使元素显示效果更好。但是如果你希望元素显示时使用第一种效果，而消失时使用第二种效果，则可以这么做：

```
// Fade
$('.btn').click(function () {
  $('.element').fadeToggle('slow');
});
// Toggle
$('.btn').click(function () {
  $('.element').slideToggle('slow');
});
```

### 9. 简单的手风琴效果

这是一个实现手风琴效果快速简单的方法：

```
// Close all panels
$('#accordion').find('.content').hide();
// Accordion
$('#accordion').find('.accordion-header').click(function () {
  var next = $(this).next();
  next.slideToggle('fast');
  $('.content').not(next).slideUp('fast');
  return false;
});
```

### 10. 让两个DIV高度相同

有时你需要让两个div度相同，而不管它们里面的内容多少。可以使用下面的代码片段：

```
var $columns = $('.column');
var height = 0;
$columns.each(function () {
  if ($(this).height() > height) {
    height = $(this).height();
  }
});
$columns.height(height);
```

这段代码会循环一组元素，并设置它们的高度为元素中的最大高。