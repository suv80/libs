---
title: 开发者必须知道的HTML5十五大新特性
tags: [html]
date: 2015/02/14
---

HTML5想必大家都很熟悉了，因为太多的媒体在讨论这一技术。然而，你能准确地说出HTML5带来了哪些新特性吗？本文总结了HTML5带来的15项你必须知道的新特性。

一起来看下：

### 1.新的文档类型 (New Doctype)

目前许多网页还在使用XHTML 1.0 并且要在第一行像这样声明文档类型：

```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd" 
```

在HTML5中，上面那种声明方式将失效。下面是HTML5中的声明方式：

```html
    <!DOCTYPE html> 
```

### 2.脚本和链接无需type  (No More Types for Scripts and Links)

在HTML4或XHTML中，你需要用下面的几行代码来给你的网页添加CSS和JavaScript文件。

```html
    <link rel="stylesheet" href="path/to/stylesheet.css" type="text/css" /> 
    <script type="text/javascript" src="path/to/script.js"></script> 
```

而在HTML5中，你不再需要指定类型属性。因此，代码可以简化如下： 

```html
    <link rel="stylesheet" href="path/to/stylesheet.css" /> 
    <script src="path/to/script.js"></script> 
```

### 3.语义Header和Footer (The Semantic Header and Footer)
在HTML4或XHTML中，你需要用下面的代码来声明“Header”和“Footer”。

```html
    <div id="header"> 
        ... 
    </div> 
    .......... 
    <div id="footer"> 
        ... 
    </div> 
```

在HTML5中，有两个可以替代上述声明的元素，这可以使代码更简洁。

```html
    <header> 
        ... 
    </header> 
    <footer> 
        ... 
    </footer> 
```

### 4.Hgroup
在HTML5中，有许多新引入的元素，hgroup就是其中之一。假设我的网站名下面紧跟着一个子标题，我可以用`<h1>`和`<h2>`标签来分别定义。然而，这种定义没有说明这两者之间的关系。而且，h2标签的使用会带来更多问题，比如该页面上还有其他标题的时候。

在HTML5中，我们可以用hgroup元素来将它们分组，这样就不会影响文件的大纲。

```html
    <header> 
        <hgroup> 
            <h1> Recall Fan Page </h1> 
            <h2> Only for people who want the memory of a lifetime. </h2> 
        </hgroup> 
    </header>
```

### 5.标记元素 (Mark Element)
你可以把它当做高亮标签。被这个标签修饰的字符串应当和用户当前的行动相关。比如说，当我在某博客中搜索“Open your Mind”时，我可以利用一些JavaScript将出现的词组用`<mark>`修饰一下。

```html
    <h3> Search Results </h3> 
    <p> They were interrupted, just after Quato said, <mark>"Open your Mind"</mark>. 
    </p> 
```

### 6.图形元素 (Figure Element)
在HTML4或XHTML中，下面的这些代码被用来修饰图片的注释。

```html
    <img src="path/to/image" alt="About image" /> 
    <p>Image of Mars. </p> 
```

然而，上述代码没有将文字和图片内在联系起来。因此，HTML5引入了`<figure>`元素。当和`<figcaption>`结合起来后，我们可以语义化地将注释和相应的图片联系起来。

```html
    <figure> 
        <img src="path/to/image" alt="About image" /> 
        <figcaption> 
            <p>This is an image of something interesting. </p> 
        </figcaption> 
    </figure> 
```

### 7.重新定义`<small>` (Small Element redefined)
在HTML4或XHTML中，`<small>`元素已经存在。然而，却没有如何正确使用这一元素的完整说明。在HTML5中，`<small>`被用来定义小字。试想下你网站底部的版权状态，根据对此元素新的HTML5定义，`<small>`可以正确地诠释这些信息。

### 8.占位符 (Placeholder)
在HTML4或XHTML中，你需要用JavaScript来给文本框添加占位符。比如，你可以提前设置好一些信息，当用户开始输入时，文本框中的文字就消失。

而在HTML5中，新的“placeholder”就简化了这个问题。

### 9.必要属性 (Required Attribute)
HTML5中的新属性“required”指定了某一输入是否必需。有两种方法声明这一属性。

```html
    <input type="text" name="someInput" required> 
    <input type="text" name="someInput" required="required"> 
```

当文本框被指定必需时，如果空白的话表格就不能提交。下面是一个如何使用的例子。

```html
    <form method="post" action=""> 
        <label for="someInput"> Your Name: </label> 
        <input type="text" id="someInput" name="someInput" placeholder="Douglas Quaid" required> 
        <button type="submit">Go</button> 
    </form>
```

在上面那个例子中，如果输入内容空且表格被提交，输入框将被高亮显示。

### 10.Autofocus 属性 (Autofocus Attribute)
同样，HTML5的解决方案消除了对JavaScript的需要。如果一个特定的输入应该是“选择”或聚焦，默认情况下，我们现在可以利用自动聚焦属性。

```html
    <input type="text" name="someInput" placeholder="Douglas Quaid" required autofocus> 
```

### 11.Audio 支持 (Audio Support)
目前我们需要依靠第三方插件来渲染音频。然而在HTML5中，<audio>元素被引进来了。

```html
    <audio autoplay="autoplay" controls="controls"> 
        <source src="file.ogg" /> 
        <source src="file.mp3" /> 
        <a href="file.mp3">Download this file.</a> 
    </audio>
```

当使用`<audio>`元素时请记得包含两种音频格式。FireFox想要.ogg格式的文件，而Webkit浏览器则需要.mp3格式的。和往常一样，IE是不支持的，且Opera 10及以下版本只支持.wav格式。

### 12.Video 支持 (Video Support)
HTML5中不仅有`<audio>`元素，而且还有`<video>`。然而，和`<audio>`类似，HTML5中并没有指定视频解码器，它留给了浏览器来决定。虽然Safari和Internet Explorer9可以支持H.264格式的视频，Firefox和Opera是坚持开源Theora 和Vorbis格式。因此，指定HTML5的视频时，你必须提供这两种格式。

```html
    <video controls preload> 
        <source src="cohagenPhoneCall.ogv" type="video/ogg; codecs='vorbis, theora'" /> 
        <source src="cohagenPhoneCall.mp4" type="video/mp4; 'codecs='avc1.42E01E, mp4a.40.2'" /> 
        <p> Your browser is old. <a href="cohagenPhoneCall.mp4">Download this video instead.</a> </p> 
    </video> 
```

### 13.视频预载 (Preload attribute in Videos element)
当用户访问页面时这一属性使得视频得以预载。为了实现这个功能，可以在`<video>`元素中加上preload=”preload”或者只是preload。

```html
    <video preload> 
```

### 14.显示控制条 (Display Controls)
如果你使用过上面的每一个提到的技术点，你可能已经注意到，使用上面的代码，视频仅仅显示的是张图片，没有控制条。为了渲染出播放控制条，我们必须在video元素内指定controls属性。

```html
    <video preload controls> 
```

### 15.正规表达式 (Regular Expressions)
在HTML4或XHTML中，你需要用一些正规表达式来验证特定的文本。而HTML5中新的pattern属性让我们能够在标签处直接插入一个正规表达式。

```html
    <form action="" method="post"> 
        <label for="username">Create a Username: </label> 
        <input type="text" name="username" id="username" placeholder="4 <> 10" pattern="[A-Za-z]{4,10}" 
            autofocus 
            required> 
        <button type="submit">Go </button> 
    </form> 
```

### 结论
事实上，还有很多新元素和特性，上面提到的只是一些我认为网站开发中常用的，剩下的就由你们自己去摸索啦。

转载至[HTML5China.com](http://www.html5china.com/course/20120225_3483.html)
