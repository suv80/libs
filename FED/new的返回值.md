---
title: new的返回值
tags: [javascript]
date: 2016/04/25
---

你将会遇到在JavaScript中使用new来分配新对象的一些情况。这将会扰乱你的思绪，除非你阅读了这篇文章并理解在内部发生了什么。

JavaScript中的new操作在合理的情况下然会一个新的对象实例。我们来看，我们有一个构造函数：

```
function Thing() {
  this.one = 1;
  this.two = 2;
}

var myThing = new Thing();

myThing.one // 1
myThing.two // 2
```

> 提示: this指向new产生的新对象。否则如果Thing()不适用new调用, 将不会生成新对象, 而且this 将会指向全局对象，也就是window。这意味着：你突然有两个全局变量one和two。

myThing现在为undefined，因为Thing()中没有返回任何东西。

现在我们又有一个例子，而它却有些让人搞不懂。我们看我在构造函数里加了一条语句：

```
function Thing() {
  this.one = 1;
  this.two = 2;

  return 5;
}

var myThing = new Thing();
```

现在myThing等于什么呢？5？一个对象？还是我受伤的自我价值观？或许永远不知道！

除了能知道：

```
myThing.one // 1
myThing.two // 2
```

很有趣，我们构造函数里返回的5怎么找不到了？这很奇怪不是吗？函数都做了什么？5呢？让我们再试试别的。

我们返回一个非原始类型试一下，比如一个对象：

```
function Thing() {
  this.one = 1;
  this.two = 2;

  return {
    three: 3,
    four: 4
  };
}

var myThing = new Thing();
```

让我们试一试。直接console.log出所有内容：

```
console.log(myThing);
/*
  Object {three: 3, four: 4}
  this.one 和 this.two发生了什么!?
  他们被覆盖了，朋友。
*/
```

我们了解到： 

当你使用new关键字调用一个函数的时候，你可以使用this关键字给其设置参数（但这些你应该已经知道了）。使用new关键字调用一个返回原始变量的函数将不会返回你指定的值，而是返回函数的实例this（你指定参数的那个对象，像 this.one = 1;).

然而，返回一个非原始变量像object、array或function将会覆盖this实例，并返回那个费原始变量，有效的破坏了你分配给this的所有工作。