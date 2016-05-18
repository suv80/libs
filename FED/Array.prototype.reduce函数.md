---
title: Array.prototype.reduce函数
tags: [javascript]
date: 2016/04/10
---

### 概述

reduce() 方法接收一个函数作为累加器（accumulator），数组中的每个值（从左到右）开始合并，最终为一个值。

### 语法

`arr.reduce(callback,[initialValue])`

**参数**

*callback 执行数组中每个值的函数，包含四个参数*

> *previousValue*
> 上一次调用回调返回的值，或者是提供的初始值（initialValue）
> *currentValue*
> 数组中当前被处理的元素
> *index*
> 当前元素在数组中的索引
> *array*
> 调用 reduce 的数组
> *initialValue*
> 作为第一次调用 callback 的第一个参数。

### 描述

reduce 为数组中的每一个元素依次执行回调函数，不包括数组中被删除或从未被赋值的元素，接受四个参数：初始值（或者上一次回调函数的返回值），当前元素值，当前索引，调用 reduce 的数组。

回调函数第一次执行时，previousValue 和 currentValue 可以是一个值，如果 initialValue 在调用 reduce 时被提供，那么第一个 previousValue 等于 initialValue ，并且currentValue 等于数组中的第一个值；如果initialValue 未被提供，那么previousValue 等于数组中的第一个值，currentValue等于数组中的第二个值。

如果数组为空并且没有提供initialValue， 会抛出TypeError 。如果数组仅有一个元素（无论位置如何）并且没有提供initialValue， 或者有提供initialValue但是数组为空，那么此唯一值将被返回并且callback不会被执行。

例如执行下面的代码

```
[0,1,2,3,4].reduce(function(previousValue, currentValue, index, array){
  return previousValue + currentValue;
});
```

回调被执行四次，每次的参数和返回值如下表：

| previousValue | currentValue | index | array | return | value       |
| ------------- | ------------ | ----- | ----- | ------ | ----------- |
| first         | call         | 0     | 1     | 1      | [0,1,2,3,4] |
| second        | call         | 1     | 2     | 2      | [0,1,2,3,4] |
| third         | call         | 3     | 3     | 3      | [0,1,2,3,4] |
| fourth        | call         | 6     | 4     | 4      | [0,1,2,3,4] |

reduce 的返回值是回调函数最后一次被调用的返回值（10）。

如果把初始值作为第二个参数传入 reduce，最终返回值变为20，结果如下：

```
[0,1,2,3,4].reduce(function(previousValue, currentValue, index, array){
  return previousValue + currentValue;
}, 10);
```

| previousValue | currentValue | index | array | return      | value |
| ------------- | ------------ | ----- | ----- | ----------- | ----- |
| 第一次调用         | 10           | 0     | 0     | [0,1,2,3,4] | 10    |
| 第二次调用         | 10           | 1     | 1     | [0,1,2,3,4] | 11    |
| 第三次调用         | 11           | 2     | 2     | [0,1,2,3,4] | 13    |
| 第四次调用         | 13           | 3     | 3     | [0,1,2,3,4] | 16    |
| 第五次调用         | 16           | 4     | 4     | [0,1,2,3,4] | 20    |

### 例子

例子:将数组所有项相加

```
var total = [0, 1, 2, 3].reduce(function(a, b) {
    return a + b;
});
// total == 6
```

例子: 数组扁平化

```
var flattened = [[0, 1], [2, 3], [4, 5]].reduce(function(a, b) {
    return a.concat(b);
});
// flattened is [0, 1, 2, 3, 4, 5]
```

### 兼容旧环境（Polyfill）

Array.prototype.reduce 被添加到 ECMA-262 标准第 5 版；因此可能在某些实现环境中不被支持。可以将下面的代码插入到脚本开头来允许在那些未能原生支持 reduce 的实现环境中使用它。

```
if ('function' !== typeof Array.prototype.reduce) {
  Array.prototype.reduce = function(callback, opt_initialValue){
    'use strict';
    if (null === this || 'undefined' === typeof this) {
      // At the moment all modern browsers, that support strict mode, have
      // native implementation of Array.prototype.reduce. For instance, IE8
      // does not support strict mode, so this check is actually useless.
      throw new TypeError(
          'Array.prototype.reduce called on null or undefined');
    }
    if ('function' !== typeof callback) {
      throw new TypeError(callback + ' is not a function');
    }
    var index, value,
        length = this.length >>> 0,
        isValueSet = false;
    if (1 < arguments.length) {
      value = opt_initialValue;
      isValueSet = true;
    }
    for (index = 0; length > index; ++index) {
      if (this.hasOwnProperty(index)) {
        if (isValueSet) {
          value = callback(value, this[index], index, this);
        }
        else {
          value = this[index];
          isValueSet = true;
        }
      }
    }
    if (!isValueSet) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    return value;
  };
}
```

### 浏览器兼容性
**Desktop**

| Feature       | Chrome | Firefox (Gecko) | Internet Explorer | Opera | Safari |
| ------------- | ------ | --------------- | ----------------- | ----- | ------ |
| Basic support | (Yes)  | 3.0(1.9)        | 9                 | 10.5  | 4.0    |

**Mobile**

| Feature       | Android | Chrome for Android | Firefox Mobile (Gecko) | IE Mobile | Opera Mobile | Safari Mobile |
| ------------- | ------- | ------------------ | ---------------------- | --------- | ------------ | ------------- |
| Basic support | (Yes)   | (Yes)              | (Yes)                  | (Yes)     | (Yes)        | (Yes)         |
