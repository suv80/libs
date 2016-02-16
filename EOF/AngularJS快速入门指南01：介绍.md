AngularJS是一个JavaScript框架。它可以通过```<script>```标记被添加到HTML页面中。

AngularJS通过指令对HTML属性进行了扩展，然后通过表达式将数据绑定到HTML元素中。

AngularJS是一个JavaScript框架，它是由JavaScript语言编写的类库。

AngularJS以JavaScript文件的形式进行发布，我们可以通过script标记将它添加到web页面中：

```
<script src="http://cdn.bootcss.com/angular.js/1.3.14/angular.min.js"></script>
```

AngularJS扩展了HTML

AngularJS通过一系列ng-directives指令对HTML进行扩展。

ng-app指令定义了AngularJS application。

ng-model指令将HTML控件的值与数据模型绑定到一起。

ng-bind指令将模型数据绑定到HTML视图。

```
<script src="http://cdn.bootcss.com/angular.js/1.3.14/angular.js"></script>
<body>

<div ng-app="">
     <p>Name: <input type="text" ng-model="name"></p>
     <p ng-bind="name"></p>
</div>

</body>
```

<a href="https://jsfiddle.net/Lionney/5gzeb661/" target="_blank">运行</a>

***示例说明：***

>当页面加载完成时AngularJS自动开始执行。

>ng-app指令告诉AngularJS它所在的<div>元素是AngularJS Application的根元素。

>ng-model指令将input标签的值绑定给变量name。

>ng-bind指令将变量name的值绑定给<p>元素的innerHTML属性。

***AngularJS指令***

就如你所看到的，AngularJS指令就是一组以ng开头的HTML属性。

通过ng-init指令可以将AngularJS Application的变量进行初始化。

```
<div ng-app="" ng-init="firstName='John'">

<p>The name is <span ng-bind="firstName"></span></p>

</div>
```

等效的代码：

```
<div data-ng-app="" data-ng-init="firstName='John'">

<p>The name is <span data-ng-bind="firstName"></span></p>

</div>
```

***Note***

你可以使用前缀data-ng-来代替ng-，这样可以确保页面上的HTML是有效的（valid）。

在后面的章节中你将会学习到更多的AngularJS指令。

***AngularJS表达式***

AngularJS表达式写在双大括号中：{{ 表达式语句 }}。

AngularJS会准确地将表达式“输出”为计算的结果，例如：

```
<!DOCTYPE html>
<html>
<script src="http://cdn.bootcss.com/angular.js/1.3.14/angular.js"></script>
<body>

<div ng-app="">
     <p>My first expression: {{ 5 + 5 }}</p>
</div>

</body>
</html>
```

<a href="https://jsfiddle.net/Lionney/etp27rmq/" target="_blank">运行</a>

AngularJS表达式绑定数据到HTML的方式与ng-bind指令的方式相同。

```
<!DOCTYPE html>
<html>
<script src="http://cdn.bootcss.com/angular.js/1.3.14/angular.js"></script>
<body>

<div ng-app="">
  <p>Name: <input type="text" ng-model="name"></p>
  <p>{{name}}</p>
</div>

</body>
</html>
```

<a href="https://jsfiddle.net/Lionney/02pp8ha4/" target="_blank">运行</a>

在后面的章节中你将会学习到更多有关AngularJS表达式的内容。

***AngularJS Application***

AngularJS模块定义了AngularJS Applications。

AngularJS控制器则控制着AngularJS Applications的行为。

ng-app指令用于指定application，而ng-controller指令则用来指定控制器。

```
<div ng-app="myApp" ng-controller="myCtrl">

First Name: <input type="text" ng-model="firstName"><br>
Last Name: <input type="text" ng-model="lastName"><br>
<br>
Full Name: {{firstName + " " + lastName}}

</div>

<script>
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope) {
    $scope.firstName= "John";
    $scope.lastName= "Doe";
});
</script>
```

<a href="https://jsfiddle.net/Lionney/jjuhvd9d/" target="_blank">运行</a>

AngularJS模块定义applications：

```
var app = angular.module('myApp', []);
```

AngularJS控制器控制AngularJS Applications的行为：

```
app.controller('myCtrl', function($scope) {
    $scope.firstName= "John";
    $scope.lastName= "Doe";
});
```

在后面的章节中你将会学习到更多有关模块和控制器的内容。

原文地址：<a href="http://www.cnblogs.com/jaxu/p/4483774.html" target="_blank">http://www.cnblogs.com/jaxu/p/4483774.html</a>