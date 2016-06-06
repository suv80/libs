---
title: JavaScript总结几个提高性能知识点
tags: [javascript]
date: 2015/05/21
---

***更快速的数据访问***

对于浏览器来说，一个标识符所处的位置越深，去读写他的速度也就越慢(对于这点，原型链亦是如此)。这个应该不难理解，简单比喻就是：杂货店离你家越远，你去打酱油所花的时间就越长。

如果我们需要在当前函数内多次用到一个变量值，那么我们可以用一个局部变量先将其存储起来，案例如下：

```
//修改前
function showLi(){
    var i = 0;
    for(;i<document.getElementsByTagName("li").length;i++){    //一次访问document
        console.log(i,document.getElementsByTagName("li")[i]);  //三次访问document
    };
};
//修改后
function showLi(){
    var li_s = document.getElementsByTagName("li");  //一次访问document
    var i = 0;
    for(;i<li_s.length;i++){
        console.log(i,li_s[i]);  //三次访问局部变量li_s
    };
};
```

***DOM操作的优化***

众所周知的，DOM操作远比javascript的执行耗性能，虽然我们避免不了对DOM进行操作，但我们可以尽量去减少该操作对性能的消耗。

让我们通过代码解释这个问题：

```
function innerLi_s(){
    var i = 0;
    for(;i<20;i++){
        document.getElementById("Num").innerHTML="A"; //进行了20次循环，每次又有2次DOM元素访问：一次读取innerHTML的值，一次写入值
    };
};
```

针对以上方法进行一次改写：

```
function innerLi_s(){
    var content ="";
    var i = 0;
    for(;i<20;i++){
        content += "A";  //这里只对js的变量循环了20次
    };
    document.getElementById("Num").innerHTML += content;  //这里值进行了一次DOM操作，又分2次DOM访问：一次读取innerHTML的值，一次写入值
};
```

***减少Dom的重绘重排版***

元素布局的改变或内容的增删改或者浏览器窗口尺寸改变都将会导致重排，而字体颜色或者背景色的修改则将导致重绘。
对于类似以下代码的操作，据说现代浏览器大多进行了优化(将其优化成1次重排版)：

```
//修改前
var el = document.getElementById("div");
el.style.borderLeft = "1px"; //1次重排版
el.style.borderRight = "2px"; //又1次重排版
el.style.padding = "5px"; //还有1次重排版
//修改后
var el = document.getElementById("div");
el.style.cssText = "border-left:1px;border-right:2px;padding:5px"; //1次重排版
```

针对多重操作，以下三种方法也可以减少重排版和重绘的次数：

1.Dom先隐藏，操作后再显示 2次重排 (临时的display:none)

2.document.createDocumentFragment() 创建文档片段处理，操作后追加到页面 1次重排

3.var newDOM = oldDOM.cloneNode(true)创建Dom副本，修改副本后oldDOM.parentNode.replaceChild(newDOM,oldDOM)覆盖原DOM 2次重排

***循环的优化***

这应该是较多人都知道的写法了，简单带过即可(后面还是用代码+注释形式说明)~

```
//修改前
var i = 0;
for(;i<arr.lengthli++){  //每次循环都需要获取数组arr的length
    console.log(arr[i]);
}
//修改后
var i = 0;
var len = arr.length;  //获取一次数组arr的length 
for(;i<len;i++){
    console.log(arr[i]);
}
//or
var i = arr.length;;
for(;i;i--){
    console.log(arr[i]);
}
```

***合理利用二进制***

如：对2取模，则偶数最低位是0，奇数最低位是0，与1进行位与操作的结果是0，奇数的最低位是1，与1进行位与操作的结果是1。

代码如下：

```
.odd{color:red}
.even{color:yellow}

<ul>
      <li>1</li>
      <li>2</li>
      <li>3</li>
      <li>4</li>
      <li>5</li>
      <li>6</li>
</ul>
```

```
var i = 0;
var lis = document.getElementsByTagName("li");
var len = lis.length;
for(;i<len;i++){
    if(i&1){
        lis[i].className = "even";
    } else{
        lis[i].className = "odd";
    }
};
```


虽说现代浏览器都已经做的很好了，但是本兽觉得这是自己对代码质量的一个追求。并且可能一个点或者两个点不注意是不会产生多大性能影响，但是从多个点进行优化后，可能产生的就会是质的飞跃了~
