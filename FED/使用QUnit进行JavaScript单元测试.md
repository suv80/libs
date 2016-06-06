---
title: 使用QUnit进行JavaScript单元测试
tags: [javascript,jquery,qunit]
date: 2015/01/31
---

### 简介

QUnit是一个强大的JavaScript单元测试框架。他可用于jQuery，jQuery UI和jQuery Mobile项目，以及任何使用JavaScript代码编写的项目的测试。

### 运行环境

+ 任何Html和JavaScript编辑器（Visual Studio 2013）
+ 从QUnit官方下载reference js和css文件

### 加入QUnit到单元测试

添加QUnit.js和QUnit.css到你要测试的HTML页面中。

```
<script src="//code.jquery.com/qunit/qunit-1.22.0.js"></script>
<link rel="stylesheet" href="https://code.jquery.com/qunit/qunit-1.22.0.css">
```

### 创建需要进行单元测试的JavaScript类

将要进行单元测试的代码放到一个单独的js文件中（Calculations.js）：

```
// Create Calculation class.
var Calculation = function () { };

// Add Addition to method to the Calculation class.
Calculation.prototype.Add = function (a, b) {
    return a + b;
};

// Add Subtraction method to the Calculation class.
Calculation.prototype.Substraction = function (a, b) {
    return a - b;
};

// Add Multiplication method to the Calculation class.
Calculation.prototype.Multiplication = function (a, b) {
    return a * b;
};

// Add Division method to the Calculation class.
Calculation.prototype.Division = function (a, b) {
    return a / b;
};
```

### 为上面的方法创建一个单元测试用例

下面的代码就是上面JavaScript方法的单元测试用例，我们同样将它放到单独的一个js文件中（UnitTest.js）：

```
// Instantiate Calculation class.
var c = new Calculation();
// Unit test for addition.
QUnit.test("Addition Test", function (assert) {   
    assert.ok(c.Add(2, 3) == "5", "Passed!");
});

// Unit test for subtraction.
QUnit.test("Substraction Test", function (assert) {
    assert.ok(c.Substraction(3, 2) == "1", "Passed!");
});

// Unit test for division.
QUnit.test("Division Test", function (assert) {
    assert.ok(c.Division(5, 5) == "1", "Passed!");
});

// Unit test for multiplication.
QUnit.test("Multiplication Test", function (assert) {
    assert.ok(c.Multiplication(5, 5) == "25", "Passed!");
});
```

###  在HTML代码中引用所有的js和css文件

在HTML代码中分别创建一个id为qunit、qunit-fixture的div标记。

```
<link rel="stylesheet" href="https://code.jquery.com/qunit/qunit-1.22.0.css">
<script src="~/Scripts/Calculations.js"></script>
<div id="qunit"></div>
<div id="qunit-fixture"></div>
<script src="//code.jquery.com/qunit/qunit-1.22.0.js"></script>
<script src="~/Scripts/UnitTest.js"></script>
```

[http://www.codeceo.com/article/qunit-javascript-unit-test.html](http://www.codeceo.com/article/qunit-javascript-unit-test.html)
