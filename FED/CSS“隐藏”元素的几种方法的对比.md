---
title: CSS“隐藏”元素的几种方法的对比
tags: [css]
date: 2015/10/26
---

一说起CSS隐藏元素，我想大部分小伙伴们都会想到的第一种方法就是设置display为none。这是最为人所熟知也是最常用的方法。我相信还有不少人想到使用设置visibility为hidden来隐藏元素，这种方式也是常用的方法，而且也有很多人知道两者的不同。除了这两种方法，本文还总结了一些比较不常用的方法，比较了这几种“隐藏”元素方法的区别和优缺点，欢迎大家交流！！

几种方法的简单介绍

首先我们分别来说说到底有哪几种隐藏元素的方法，有一些方法是众所周知的，还有一些算是一种技巧。

`display:none`

### 设置元素的display为none是最常用的隐藏元素的方法。

```
.hide {
     display:none;
}
```

将元素设置为`display:none`后，元素在页面上将彻底消失，元素本来占有的空间就会被其他元素占有，也就是说它会导致浏览器的重排和重绘。

```
visibility:hidden
```

设置元素的visibility为hidden也是一种常用的隐藏元素的方法，和`display:none`的区别在于，元素在页面消失后，其占据的空间依旧会保留着，所以它只会导致浏览器重绘而不会重排。

```
.hidden{
   visibility:hidden
}
```
`visibility:hidden`适用于那些元素隐藏后不希望页面布局会发生变化的场景

```
opacity:0
```

opacity属性我相信大家都知道表示元素的透明度，而将元素的透明度设置为0后，在我们用户眼中，元素也是隐藏的，这算是一种隐藏元素的方法。

```
.transparent {
   opacity:0;
}
```

这种方法和`visibility:hidden`的一个共同点是元素隐藏后依旧占据着空间，但我们都知道，设置透明度为0后，元素只是隐身了，它依旧存在页面中。

### 设置height，width等盒模型属性为0

这是我总结的一种比较奇葩的技巧，简单说就是将元素的margin，border，padding，height和width等影响元素盒模型的属性设置成0，如果元素内有子元素或内容，还应该设置其`overflow:hidden`来隐藏其子元素，这算是一种奇技淫巧。

```
.hiddenBox {
   margin:0;
   border:0;
   padding:0;
   height:0;
   width:0;
   overflow:hidden;
}
```

这种方式既不实用，也可能存在着着一些问题。但平时我们用到的一些页面效果可能就是采用这种方式来完成的，比如jquery的slideUp动画，它就是设置元素的`overflow:hidden`后，接着通过定时器，不断地设置元素的height，margin-top，margin-bottom，border-top，border-bottom，padding-top，padding-bottom为0，从而达到slideUp的效果。

### 元素隐藏后的事件响应

如果被隐藏的元素绑定了一些事件，我们执行了相关操作后，这些事件是否会被响应并执行呢，看看下面的代码：

```
<style>
    div { 
        width: 100px; 
        height: 100px; 
        background: red; 
        margin: 15px; 
        padding: 10px; 
        border: 5px solid green; 
        display: inline-block; 
        overflow: hidden; 
    }
    .none { display: none; }
    .hidden { visibility: hidden; }
    .opacity0 { opacity: 0; }
    .height0 { height: 0; }  
</style>  

<div class="none"></div>
<div class="hidden"></div>
<div class="opacity0"></div>
<div class="height0">aa</div>  

<script src="/Scripts/jquery-1.10.2.min.js"></script>
<script>
    $(".none").on("click", function () {
        console.log("none clicked");
    })
    $(".hidden").on("click", function () {
        console.log("hidden clicked");
    })
    $(".opacity0").on("click", function () {
        console.log("opacity0 clicked");
    })
    $(".height0").on("click", function () {
        console.log("height0 clicked");
    })
</script>
```

这段代码将四种隐藏元素的方法分别展示出来，然后绑定其点击事件，经过测试，主要有下面的结论：

1、`display:none`：元素彻底消失，很显然不会触发其点击事件

2、`visibility:hidden`：无法触发其点击事件，有一种说法是`display:none`是元素看不见摸不着，而`visibility:hidden`是看不见摸得着，这种说法是不准确的，设置元素的visibility后无法触发点击事件，说明这种方法元素也是消失了，只是依然占据着页面空间。

3、`opacity:0`：可以触发点击事件，原因也很简单，设置元素透明度为0后，元素只是相对于人眼不存在而已，对浏览器来说，它还是存在的，所以可以触发点击事件

4、`height:0`：将元素的高度设置为0，并且设置overflow:hidden。使用这种方法来隐藏元素，是否可以触发事件要根据具体的情况来分析。如果元素设置了border，padding等属性不为0，很显然，页面上还是能看到这个元素的，触发元素的点击事件完全没有问题。如果全部属性都设置为0，很显然，这个元素相当于消失了，即无法触发点击事件。

但是这些结论真的准确吗？
我们在上面的代码中添加这样一句代码：

```
$(".none").click();
```

结果发现，触发了click事件，也就是通过JS可以触发被设置为`display:none`的元素的事件。
所以前面无法触发点击事件的真正原因是鼠标无法真正接触到被设置成隐藏的元素！！！

### CSS3 transition对这几种方法的影响

CSS3提供的transition极大地提高了网页动画的编写，但并不是每一种CSS属性都可以通过transition来进行动画的。我们修改代码如下：

```
<style>
    div { 
        width: 100px; 
        height: 100px; 
        background: red; 
        margin: 15px; 
        padding: 10px; 
        border: 5px solid green; 
        display: inline-block; 
        overflow: hidden; 
        transition: all linear 2s;  
    }
</style>  

<div class="none"></div>
<div class="hidden"></div>
<div class="opacity0"></div>
<div class="height0">aa</div>  

<script src="/Scripts/jquery-1.10.2.min.js"></script>
<script>
$(".none").on("click", function () {
    console.log("none clicked");
    $(this).css("display", "none");
})
$(".hidden").on("click", function () {
    console.log("hidden clicked");
    $(this).css("visibility", "hidden");
})
$(".opacity0").on("click", function () {
    console.log("opacity0 clicked");
    $(this).css("opacity", 0);
})
$(".height0").on("click", function () {
    console.log("height0 clicked");
    $(this).css({
        "height": 0,
    });
})
</script>
```

经过测试，可以看到：
1、`display:none`：完全不受transition属性的影响，元素立即消失
2、`visibility:hidden`：元素消失的时间跟transition属性设置的时间一样，但是没有动画效果
3、opacity和height等属性能够进行正常的动画效果

假设我们要通过CSS3来做一个淡出的动画效果，应该如下：

```
.fadeOut { visibility: visible; opacity: 1; transition: all linear 2s; }
.fadeOut:hover { visibility: hidden; opacity: 0; }
```

应该同时设置元素的visibility和opacity属性。

### 总结说明

本文总结说明了“隐藏”元素的几种方式，其中最常用的还是`display:none`和`visibility:hidden`。其他的方式只能算是奇技淫巧，并不推荐使用它们来隐藏元素，它们的真正用途应该不在隐藏元素，而是通过了解这些方法的特点，挖掘出其真正的使用场景。欢迎大家交流！！

### 补充

来自评论区小伙伴们补充的技巧：

1、设置元素的position与left，top，bottom，right等，将元素移出至屏幕外

2、设置元素的position与z-index，将z-index设置成尽量小的负数
