---
title: 使用ajax和history.pushState无刷新改变页面URL
tags: [javascript,ajax]
date: 2015/02/04
---

如果你使用chrome或者firefox等浏览器访问本博客、github.com、plus.google.com等网站时，细心的你会发现页面之间的点击是通过ajax异步请求的，同时页面的URL发生了了改变。并且能够很好的支持浏览器前进和后退。

是什么有这么强大的功能呢？

### 表现

HTML5里引用了新的API，history.pushState和history.replaceState，就是通过这个接口做到无刷新改变页面URL的。

### 与传统的AJAX的区别

传统的ajax有如下的问题：

1、可以无刷新改变页面内容，但无法改变页面URL

2、为了更好的可访问性，内容发生改变后，通常改变URL的hash

3、hash的方式不能很好的处理浏览器的前进、后退等问题

4、进而浏览器引入了onhashchange的接口，不支持的浏览器只能定时去判断hash是否改变

5、但这种方式对搜索引擎很不友好

6、twitter和google约定了使用#!xxx（即hash第一个字符为!），搜索引擎进行支持。

为了解决传统ajax带来的问题，HTML5里引入了新的API，即：history.pushState, history.replaceState

可以通过pushState和replaceState接口操作浏览器历史，并且改变当前页面的URL。

pushState是将指定的URL添加到浏览器历史里，replaceState是将指定的URL替换当前的URL。

### 如何使用

```
var state = {
    title: title,
    url: options.url,
    otherkey: othervalue
};
window.history.pushState(state, document.title, url);
```

state对象除了要title和url之外，你也可以添加其他的数据，比如：还想将一些发送ajax的配置给保存起来。

replaceState和pushState是相似的，这里就不多介绍了。

### 如何响应浏览器的前进、后退操作

window对象上提供了onpopstate事件，上面传递的state对象会成为event的子对象，这样就可以拿到存储的title和URL了。

```
window.addEventListener('popstate', function(e){
  if (history.state){
    var state = e.state;
    //do something(state.url, state.title);
  }
}, false);
```

这样就可以结合ajax和pushState完美的进行无刷新浏览了。

### 一些限制

1、传递的URL必须是同域下的，无法跨域

2、state对象虽然可以存储很多自定义的属性，但对于不可序列化的对象则不能存储，如：DOM对象。

### 对应后端的一些处理

这种模式下除了当前使用ajax可以无刷新浏览外，还要保证直接请求改变的URL后也可以正常浏览，所以后端要对这些处理下。

1、对使用pushState的ajax发送一个特殊的头，如： setRequestHeader('PJAX', 'true')。

2、后端获取到有PJAX=true的header时，将页面中通用的部分都不输出。比如：PHP可以通过下面的判断

```
function is_pjax(){
    return array_key_exists('HTTP_X_PJAX', $_SERVER) && $_SERVER['HTTP_X_PJAX'] === 'true';
}
```

虽然接口上只有pushState、replaceState、onpopstate，但在使用的时候需要做很多的处理。

针对这个已经写好了一个基于jquery的插件，接下来的文章会详细介绍如何使用pjax(ajax+pushState)进行无刷新改变URL浏览。

@update - 2012.03.06

已经将ajax+history.pushState封装成pjax, 项目地址： [https://github.com/welefen/pjax](https://github.com/welefen/pjax)， 目前支持jquery, qwrap, kissy 3个版本

文章源：[http://www.welefen.com/use-ajax-and-pushstate.html](http://www.welefen.com/use-ajax-and-pushstate.html)
