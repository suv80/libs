---
title: 学习并运用JavaScript的原生函数
tags: [javascript]
date: 2015/01/27
---

尽管 JavaScript 总是让人产生误解，但是它已经成为了最流行的编程语言之一。理解 JavaScript 的内在原理很困难。同样的，迫使 JavaScript 成为常规规范，如面向对象或函数编程，同样具有挑战性。这里我强调阐明 JavaScript 核心部分的原生函数。

在这篇文章中，我将讨论以下几种行为：

- Call/Apply
- Bind
- Map
- Filter

首先我会定义这个函数(利用Mozilla的声明方式)，然后提供一个例子，最后实现此函数。

为了解释这些行为，我需要先解释一下复杂的 this 关键字以及类似数组的 arguments 对象。

### this 和 arguments 对象

JavaScript 的作用域是基于函数而言的，术语一般称为作用域，变量和方法的作用域都是当前函数。此外，函数执行的作用域是他们被定义的作用域而不是执行的作用域。如果你想了解更多有关于作用域的知识，可以参考你应该知道的4种 JavaScript 设计模式这篇文章。this对象引用当前函数的上下文并且可以以多种方式被调用。例如，它可以被绑定到 window 对象（全局作用域）。

```javascript
this.globalVar = {
    myGlobalVarsMethod: function (){
        // Implementation
    }
};

console.log(this.globalVar); // { myGlobalVarsMethod: [Function] }
```

并且变量可以绑定到已存在的函数中，如下：

```javascript
this.globalVariable = 'globalVariable';
function globalFunction (){
    this.innerVariable = 'innerVariable';
    console.log(this.globalVariable === undefined); // false
    console.log(this.innerVariable === 'innerVariable'); // true
    return {
        innerFunction: function () {
            console.log(this.globalVariable === undefined); // true
            console.log(this.innerVariable === undefined); // true
        }
    }
}

globalFunction().innerFunction();
```

这里存在被绑定到每一个调用函数的 this 对象。严格模式下，如果变量未定义就会抛出异常/错误( TypeErrors )。在生产环境下严格模式者会被优先考虑；然而，我故意选择不使用此模式以避免抛出异常。下面是严格模式下的一个简单例子：

```javascript
this.globalVar = 'globalVar';
function nonStrictFunctionTest () {
    return function () {
        console.log(this.globalVar); // globalVar
    }
}

function strictFunctionTest () {
    'use strict'; // Strict Mode
    return function () {
        console.log(this.globalVar); // TypeError: Cannot read property 'globalVar' of undefined
    }
}

nonStrictFunctionTest()();
strictFunctionTest()();
```

可能很多 JavaScript 开发人员不知道，创建函数时会有一个arguments对象。这是一个类似数组的对象（仅具有属性的长度）。arguments主要有三个属性，即callee(调用方法)，length，和caller（调用函数的参考）。

在一个函数中声明变量参数会替换/覆盖原先的参数对象。

如下列出的一些参数对象：

```javascript
function fn (){
    console.log(typeof arguments); // [object Object]
    console.log(arguments[0]); // DeathStar
    console.log(arguments[1]); // Tatooine
    arguments.push("Naboo"); // TypeError: undefined is not a function
    var arguments = "Star Wars";
    console.log(arguments[5]); // W
}

fn("DeathStar", "Tatooine");
```

按照如下所示，用 arguments 创建一个数组：

```javascript
var args = Array.prototype.slice.call(arguments);
Call/Apply
```

无论 call 还是 apply 都是调用对象的一个方法。关于使用点操作符，call 和 apply 都接受其作为第一个参数。如上所述，每一个函数都保持在其所定义的特定作用域内。因此，当你调用对象时必须考虑到函数的作用域。

Mozilla 浏览器的apply和call调用声明如下所示：

```javascript
fun.apply(thisArg, [argsArray])
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

通过传递 thisArg 参数，在特定的上下文中，被调用的函数可以访问或修改对象。下面的例子阐明了 call 的使用。

```javascript
this.lightSaberColor = 'none';
var darthVader = {
    team: 'Empire',
    lightSaberColor: 'Red'
};

var printLightSaberColor = function(){
    console.log(this.lightSaberColor);
}

printLightSaberColor() // none
printLightSaberColor.call(darthVader); // Red
printLightSaberColor.apply(darthVader); // Red
```

注意：第一次调用默认为全局作用域(window)，然而，第二次为 darthvader。

call 和 apply 主要的区别在于他们的声明方式不同。call 需要参数分开传递，而 apply 需要传入由参数组成的数组。我是这样记忆的：“Apply uses an Array。”当你的程序无关乎参数数目时，apply 方法可能会更加适用。

Currying(柯里化)(部分函数应用)是应用 call 和 apply 的一个函数式编程。Currying 允许我们创建返回已知条件的函数。这里是一个 currying 函数：

```javascript
var curry = function(fun) {
  // nothing to curry. return function
  if (arguments.length < 1) {
    return this;
  }

  // Create an array with the functions arguments
  var args = Array.prototype.slice.call(arguments, 1);
  return function() {
    // Apply fn with fn's arguments
    return fun.apply(this, args.concat(Array.prototype.slice.call(arguments, 0)));
  };
};

// Creating function that already predefines adding 1 to a
function addOneToNumber(a) {
    console.log(1 + a);
}

// addOneCurried is of function
var addOneCurried = curry(addOneToNumber);
console.log(addOneCurried(10)); // 11
```

虽然 arguments 不是数组，但是 Array.prototype.slice可以将类数组的对象转换成新数组。

### Bind

bind方法用于明确指定调用 this 方法。在作用域方面，类似于 call 和 apply 。当你将一个对象绑定到一个函数的 this对象时，你就会用到 bind。

如下是bind的声明：

```javascript
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

通俗地说，我们是通过 bind 向函数 fun 传递 thisArg 参数。实质上就是每次 fun 函数都必须通过传递 thisArg 参数调用 bind 方法。让我们在一个简单的例子中仔细看看。

```javascript
var lukeSkywalker = {
    mother: 'Padme Amidala',
    father: 'Anakin Skywalker'.
}

var getFather = function(){
    console.log(this.father);
}

getFather(); // undefined
getFather.bind(lukeSkywalker)(); // Anakin Skywalker
getFather(lukeSkywalker); // undefined
```

第一个getfather()返回值为 undefined 是因为在这里 father 属性没有被定义。那这时 this 代表什么呢？只要我们不明确的指定它，它就代表 window 的全局对象。第二个getfather()返回 “Anakin Skywalker”是因为getfather()中的 this 指代的是 lukeskywalker。许多Java/C++ 开发人员会设想最后一个getfather()的调用将返回预想的结果–虽然再次返回全局对象。

如下这里是 bind 的实现原理：

```javascript
Function.prototype.bind = function(scope) {
  var _that = this;
  return function() {
    return _that.apply(scope, arguments);
  }
}
```

这里 JavaScript 的作用域是合乎逻辑的，返回函数的 this 对象是不同于 bind 的 this 对象的。因此，将 this 暂时缓存给变量 _that 保证了其正确的作用域范围。否则，this.apply(scope,arguments) 将会未定义。

### Map

JavaScript 的 map 函数是遍历数组，同时转换每个元素的函数编程技术。它用 modified 元素创建了一个新数组并以回调的方式返回。关于我提到的修改或转换元素，实践表明，如果元素是对象(而不是原语),这只是克隆对象并不是从物理上改变了原生的。

以下是该方法的声明：

```javascript
arr.map(callback[, thisArg])
```

回调方法有三个参数，即 currentValue，index，和 array。

这里是一个有关于 map 的简单例子：

````javascript
function Jedi(name) {
    this.name = name;
}

var kit = new Jedi('Kit');
var count = new Jedi('Count');
var mace = new Jedi('Mace');
var jedis = [kit, count, mace];
var lastNames = ['Fisto', 'Dooku', 'Windu'];
var jedisWithFullNames = jedis.map(function(currentValue, index, array) {
    var clonedJedi = (JSON.parse(JSON.stringify(currentValue))) // Clone currentValue
    clonedJedi.name = currentValue.name + " " + lastNames[index];
    return clonedJedi;
});

jedisWithFullNames.map(function(currentValue) {
    console.log(currentValue.name);
});

/**
Output:
Kit Fisto
Count Dooku
Mace Windu
*/
````

了解了 map 是用来做什么的，让我们看一下它具体是如何实现的：

```javascript
Array.prototype.map = function (fun, thisArg) {
    if(typeof fun !== 'function') {
        throw new Error("The first argument must be of type function");
    }
  
    var arr = [];
    thisArg = (thisArg) ? thisArg : this;
    thisArg.forEach(function(element) {
        arr[arr.length] = fun.call(thisArgs, element);
    });

    return arr;
}
```

注：这是一个简单的实现。到 ECMAScript 5看全部的实现，并查阅其规范。

### Filter

filter 方法是数组的另外一种表现行为。类似于 map，filter 返回一个新的数组并接受一个函数和一个可选的 thisArg 参数。然而，返回的数组仅包含适合在回调函数测试的特定条件的元素。回调函数必须返回一个 Boolean –返回 true 的元素才会被接受并插入到返回的数组。

关于 filter 有许多应用，包括选择偶数，用一个特定的属性选择对象，或选择有效的电话号码。

这里是其中一种声明方法：

```javascript
arr.filter(callback[, thisArg])
```

同样的，thisArg 是可选的参数并且回调函数接受三个参数，currentValue，index 和 array。

这里是一个有关于 filter 的例子：

```javascript
function Person(name, side) {
    this.name = name;
    this.side = side;
}

var hanSolo = new Person('Han Solo','Rebels');
var bobaFett = new Person('Boba Fett','Empire');
var princessLeia = new Person('Princess Leia', 'Rebels');
var people = [hanSolo, bobaFett, princessLeia];
var enemies = people.filter(function (currentValue, index, array) {
    return currentValue.side === 'Empire';
})

.map(function(currentValue) {
    console.log(currentValue.name + " fights for the " + currentValue.side + ".");
});

/**
Output:
Boba Fett fights for the Empire.
*/
```

有趣的是，array 方法可以创造有趣的，复杂的操作。

最后，让我们看看 filter 的实现：

```javascript
Array.prototype.filter = function(fun, thisArg) {
    if(typeof fun !== 'function') {
        throw new Error("The first argument must be of type function");
    }

    var arr = [];
    thisArg = (thisArg) ? thisArg : this;
    thisArg.forEach(function(element) {
      if (fun.call(thisArg, element)) {
        arr[arr.length] = element;
      }
    });

    return arr;
};
```

这里是 ECMAScript 的实现规范。

### 总结

还有更多令人困惑但是很有用的原生函数。它们是值得用数组和函数来回顾其中的每一种方法。

希望这篇文章可以有助于你理解 JavaScript 的内部原理和词法作用域。尽管与实践紧密相连，call、apply，和bind 还是很难把握的。为了避免传统的循环技术你可以尝试使用 map 和 filter 方法 。

> 作者：Devan Patel
>
> 翻译自：https://scotch.io/tutorials/learning-javascript-native-functions-and-how-to-use-them
>
> 感谢：[typora](http://www.typora.io/)

