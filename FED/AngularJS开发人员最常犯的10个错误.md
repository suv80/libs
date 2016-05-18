---
title: AngularJS开发人员最常犯的10个错误
tags: [javascript,angularjs]
date: 2016/04/17
---

### 简介

AngularJS是目前最为活跃的Javascript框架之一，AngularJS的目标之一是简化开发过程，这使得AngularJS非常善于构建小型app原型，但AngularJS对于全功能的客户端应用程序同样强大，它结合了开发简便，特性广泛和出众的性能，使其被广泛使用。然而，大量使用也会产生诸多误区。以下这份列表摘取了常见的一些AngularJS的错误用法，尤其是在app开发过程中。

### 1. MVC目录结构

AngularJS，直白地说，就是一个MVC框架。它的模型并没有像backbone.js框架那样定义的如此明确，但它的体系结构却恰如其分。当你工作于一个MVC框架时，普遍的做法是根据文件类型对其进行归类：

```
templates/
    _login.html
    _feed.html
app/
    app.js
    controllers/
        LoginController.js
        FeedController.js
    directives/
        FeedEntryDirective.js
    services/
        LoginService.js
        FeedService.js
    filters/
        CapatalizeFilter.js
```

看起来，这似乎是一个显而易见的结构，更何况Rails也是这么干的。然而一旦app规模开始扩张，这种结构会导致你一次需要打开很多目录，无论你是使用sublime，Visual Studio或是Vim结合Nerd Tree，你都会投入很多时间在目录树中不断地滑上滑下。

与按照类型划分文件不同，取而代之的，我们可以按照特性划分文件：

```
app/
    app.js
    Feed/
        _feed.html
        FeedController.js
        FeedEntryDirective.js
        FeedService.js
    Login/
        _login.html
        LoginController.js
        LoginService.js
    Shared/
        CapatalizeFilter.js
```

这种目录结构使得我们能够更容易地找到与某个特性相关的所有文件，继而加快我们的开发进度。尽管将.html和.js文件置于一处可能存在争议，但节省下来的时间更有价值。

### 2. 模块

将所有东西都一股脑放在主模块下是很常见的，对于小型app，刚开始并没有什么问题，然而很快你就会发现坑爹的事来了。

```
var app = angular.module('app',[]);
app.service('MyService', function(){
    
//service code
});
app.controller('MyCtrl', function($scope, MyService){
    
//controller code
});
```

在此之后，一个常见的策略是对相同类型的对象归类。

```
var services = angular.module('services',[]);
services.service('MyService', function(){
    
//service code
});
 
var controllers = angular.module('controllers',['services']);
controllers.controller('MyCtrl', function($scope, MyService){
    
//controller code
});
 
var app = angular.module('app',['controllers', 'services']);
```

这种方式和前面第一部分所谈到的目录结构差不多：不够好。根据相同的理念，可以按照特性归类，这会带来可扩展性。

```
var sharedServicesModule = angular.module('sharedServices',[]);
sharedServices.service('NetworkService', function($http){});
 
var loginModule = angular.module('login',['sharedServices']);
loginModule.service('loginService', function(NetworkService){});
loginModule.controller('loginCtrl', function($scope, loginService){});
 
var app = angular.module('app', ['sharedServices', 'login']);
```

当我们开发一个大型应用程序时，可能并不是所有东西都包含在一个页面上。将同一类特性置于一个模块内，能使跨app间重用模块变得更容易。

### 3. 依赖注入

依赖注入是AngularJS最好的模式之一，它使得测试更为简单，并且依赖任何指定对象都很明确。AngularJS的注入方式非常灵活，最简单的方式只需要将依赖的名字传入模块的function中即可：

```
var app = angular.module('app',[]);
 
app.controller('MainCtrl', function($scope, $timeout){
    $timeout(function(){
        console.log($scope);
    }, 1000);
});
```

这里，很明显，MainCtrl依赖$scope和$timeout。

直到你准备将其部署到生产环境并希望精简代码时，一切都很美好。如果使用UglifyJS，之前的例子会变成下面这样：

```
var app=angular.module("app",[]);
app.controller("MainCtrl",function(e,t){t(function(){console.log(e)},1e3)})
```

现在AngularJS怎么知道MainCtrl依赖谁？AngularJS提供了一种非常简单的解决方法，即将依赖作为一个数组传入，数组的最后一个元素是一个函数，所有的依赖项作为它的参数。

```
app.controller('MainCtrl', ['$scope', '$timeout', function($scope, $timeout){
    $timeout(function(){
        console.log($scope);
    }, 1000);
}]);
```

这样做能够精简代码，并且AngularJS知道如何解释这些明确的依赖：

```
app.controller("MainCtrl",["$scope","$timeout",function(e,t){t(function(){console.log(e)},1e3)}])
```

**3.1 全局依赖**

在编写AngularJS程序时，时常会出现这种情况：某个对象有一个依赖，而这个对象又将其自身绑定在全局scope上，这意味着在任何AngularJS代码中这个依赖都是可用的，但这却破坏了依赖注入模型，并会导致一些问题，尤其体现在测试过程中。

使用AngularJS可以很容易的将这些全局依赖封装进模块中，所以它们可以像AngularJS标准模块那样被注入进去。

Underscrore.js是一个很赞的库，它可以以函数式的风格简化Javascript代码，通过以下方式，你可以将其转化为一个模块：

```
var underscore = angular.module('underscore', []);
underscore.factory('_', function() {
  return window._; 
//Underscore must already be loaded on the page
});
var app = angular.module('app', ['underscore']);
 
app.controller('MainCtrl', ['$scope', '_', function($scope, _) {
    init = function() {
          _.keys($scope);
      }
 
      init();
}]);
```

这样的做法允许应用程序继续以AngularJS依赖注入的风格进行开发，同时在测试阶段也能将underscore交换出去。

这可能看上去十分琐碎，没什么必要，但如果你的代码中正在使用use strict（而且必须使用），那这就是必要的了。

### 4. 控制器膨胀

控制器是AngularJS的肉和土豆，一不小心就会将过多的逻辑加入其中，尤其是刚开始的时候。控制器永远都不应该去操作DOM，或是持有DOM选择器，那是我们需要使用指令和ng-model的地方。同样的，业务逻辑应该存在于服务中，而非控制器。

数据也应该存储在服务中，除非它们已经被绑定在$scope上了。服务本身是单例的，在应用程序的整个生命周期都存在，然而控制器在应用程序的各状态间是瞬态的。如果数据被保存在控制器中，当它被再次实例化时就需要重新从某处获取数据。即使将数据存储于localStorage中，检索的速度也要比Javascript变量慢一个数量级。

AngularJS在遵循单一职责原则（SRP）时运行良好，如果控制器是视图和模型间的协调者，那么它所包含的逻辑就应该尽量少，这同样会给测试带来便利。

### 5. Service vs Factory

几乎每一个AngularJS开发人员在初学时都会被这些名词所困扰，这真的不太应该，因为它们就是针对几乎相同事物的语法糖而已！

以下是它们在AngularJS源代码中的定义：

```
function factory(name, factoryFn) { 
    return provider(name, { $get: factoryFn }); 
}
 
function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
}
```

从源代码中你可以看到，service仅仅是调用了factory函数，而后者又调用了provider函数。事实上，AngularJS也为一些值、常量和装饰提供额外的provider封装，而这些并没有导致类似的困惑，它们的文档都非常清晰。

由于service仅仅是调用了factory函数，这有什么区别呢？线索在$injector.instantiate：在这个函数中，$injector在service的构造函数中创建了一个新的实例。

以下是一个例子，展示了一个service和一个factory如何完成相同的事情：

```
var app = angular.module('app',[]);
 
app.service('helloWorldService', function(){
    this.hello = function() {
        return "Hello World";
    };
});
 
app.factory('helloWorldFactory', function(){
    return {
        hello: function() {
            return "Hello World";
        }
    }
});
```

当helloWorldService或helloWorldFactory被注入到控制器中，它们都有一个hello方法，返回”hello world”。service的构造函数在声明时被实例化了一次，同时factory对象在每一次被注入时传递，但是仍然只有一个factory实例。所有的providers都是单例。

既然能做相同的事，为什么需要两种不同的风格呢？相对于service，factory提供了更多的灵活性，因为它可以返回函数，这些函数之后可以被新建出来。这迎合了面向对象编程中工厂模式的概念，工厂可以是一个能够创建其他对象的对象。

```
app.factory('helloFactory', function() {
    return function(name) {
        this.name = name;
 
        this.hello = function() {
            return "Hello " + this.name;
        };
    };
});
```

这里是一个控制器示例，使用了service和两个factory，helloFactory返回了一个函数，当新建对象时会设置name的值。

```
app.controller('helloCtrl', function($scope, helloWorldService, helloWorldFactory, helloFactory) {
    init = function() {
      helloWorldService.hello(); 
//'Hello World'
      helloWorldFactory.hello(); 
//'Hello World'
      new helloFactory('Readers').hello() 
//'Hello Readers'
    }
 
    init();
});
```

在初学时，最好只使用service。

Factory在设计一个包含很多私有方法的类时也很有用：

```
app.factory('privateFactory', function(){
    var privateFunc = function(name) {
        return name.split("").reverse().join(""); 
//reverses the name
    };
 
    return {
        hello: function(name){
          return "Hello " + privateFunc(name);
        }
    };
});
```

通过这个例子，我们可以让privateFactory的公有API无法访问到privateFunc方法，这种模式在service中是可以做到的，但在factory中更容易。

### 6. 没有使用Batarang

Batarang是一个出色的Chrome插件，用来开发和测试AngularJS app。

Batarang提供了浏览模型的能力，这使得我们有能力观察AngularJS内部是如何确定绑定到作用域上的模型的，这在处理指令以及隔离一定范围观察绑定值时非常有用。

Batarang也提供了一个依赖图， 如果我们正在接触一个未经测试的代码库，这个依赖图就很有用，它能决定哪些服务应该被重点关照。

最后，Batarang提供了性能分析。Angular能做到开包即用，性能良好，然而对于一个充满了自定义指令和复杂逻辑的应用而言，有时候就不那么流畅了。使用Batarang性能工具，能够直接观察到在一个digest周期中哪个函数运行了最长时间。性能工具也能展示一棵完整的watch树，在我们拥有很多watcher时，这很有用。

### 7. 过多的watcher

在上一点中我们提到，AngularJS能做到开包即用，性能良好。由于需要在一个digest周期中完成脏数据检查，一旦watcher的数量增长到大约2000时，这个周期就会产生显著的性能问题。（2000这个数字不能说一定会造成性能大幅下降，但这是一个不错的经验数值。在AngularJS 1.3 release版本中，已经有一些允许严格控制digest周期的改动了，Aaron Gray有一篇很好的文章对此进行解释。）

以下这个“立即执行的函数表达式(IIFE)”会打印出当前页面上所有的watcher的个数，你可以简单的将其粘贴到控制台中，观察结果。这段IIFE来源于Jared在StackOverflow上的回答：

```
(function () { 
    var root = $(document.getElementsByTagName('body'));
    var watchers = [];
 
    var f = function (element) {
        if (element.data().hasOwnProperty('$scope')) {
            angular.forEach(element.data().$scope.$$watchers, function (watcher) {
                watchers.push(watcher);
            });
        }
 
        angular.forEach(element.children(), function (childElement) {
            f($(childElement));
        });
    };
 
    f(root);
 
    console.log(watchers.length);
})();
```

通过这个方式得到watcher的数量，结合Batarang性能板块中的watch树，应该可以看到哪里存在重复代码，或着哪里存在不变数据同时拥有watch。

当存在不变数据，而你又想用AngularJS将其模版化，可以考虑使用bindonce。Bindonce是一个简单的指令，允许你使用AngularJS中的模版，但它并不会加入watch，这就保证了watch数量不会增长。

### 8. 限定$scope的范围

Javascript基于原型的继承与面向对象中基于类的继承有着微妙的区别，这通常不是什么问题，但这个微妙之处在使用$scope时就会表现出来。在AngularJS中，每个$scope都会继承父$scope，最高层称之为$rootScope。（$scope与传统指令有些不同，它们有一定的作用范围i，且只继承显式声明的属性。）

由于原型继承的特点，在父类和子类间共享数据不太重要，不过如果不小心的话，也很容易误用了一个父$scope的属性。

比如说，我们需要在一个导航栏上显示一个用户名，这个用户名是在登录表单中输入的，下面这种尝试应该是能工作的：

```
<div ng-controller="navCtrl">
   <span>{{user}}</span>
   <div ng-controller="loginCtrl">
        <span>{{user}}</span>
        <input ng-model="user"></input>
   </div>
</div>
```

那么问题来了……：在text input中设置了user的ng-model，当用户在其中输入内容时，哪个模版会被更新？navCtrl还是loginCtrl，还是都会？

如果你选择了loginCtrl，那么你可能已经理解了原型继承是如何工作的了。

当你检索字面值时，原型链并不起作用。如果navCtrl也同时被更新的话，检索原型链是必须的；但如果值是一个对象，这就会发生。（记住，在Javascript中，函数、数组和对象都是对象）

所以为了获得预期的行为，需要在navCtrl中创建一个对象，它可以被loginCtrl引用。

```
<div ng-controller="navCtrl">
   <span>{{user.name}}</span>
   <div ng-controller="loginCtrl">
        <span>{{user.name}}</span>
        <input ng-model="user.name"></input>
   </div>
</div>
```

现在，由于user是一个对象，原型链就会起作用，navCtrl模版和$scope和loginCtrl都会被更新。

这看上去是一个很做作的例子，但是当你使用某些指令去创建子$scope，如ngRepeat时，这个问题很容易就会产生。

### 9. 手工测试

由于TDD可能不是每个开发人员都喜欢的开发方式，因此当开发人员检查代码是否工作或是否影响了其它东西时，他们会做手工测试。

不去测试AngularJS app，这是没有道理的。AngularJS的设计使得它从头到底都是可测试的，依赖注入和ngMock模块就是明证。AngularJS核心团队已经开发了众多能够使测试更上一层楼的工具。

**9.1 Protractor**

单元测试是一个测试工作的基础，但考虑到app的日益复杂，集成测试更贴近实际情况。幸运的是，AngularJS的核心团队已经提供了必要的工具。

我们已经建立了Protractor，一个端到端的测试器用以模拟用户交互，这能够帮助你验证你的AngularJS程序的健康状况。

Protractor使用Jasmine测试框架定义测试，Protractor针对不同的页面交互行为有一个非常健壮的API。

我们还有一些其他的端到端测试工具，但是Protractor的优势是它能够理解如何与AngularJS代码协同工作，尤其是在$digest周期中。

**9.2 Karma**

一旦我们用Protractor完成了集成测试的编写工作，接下去就是执行测试了。等待测试执行，尤其是集成测试，对每个开发人员都是一种淡淡的忧伤。AngularJS的核心团队也感到极为蛋疼，于是他们开发了Karma。

Karma是一个测试器，它有助于关闭反馈回路。Karma之所以能够做到这点，是因为它在指定文件被改变时就运行测试。Karma同时也会在多个浏览器上运行测试，不同的设备也可以指向Karma服务器，这样就能够更好地覆盖真实世界的应用场景。

### 10. 使用jQuery

jQuery是一个酷炫的库，它有标准化的跨平台开发，几乎已经成为了现代化Web开发的必需品。不过尽管JQuery如此多的优秀特性，它的理念和AngularJS并不一致。

AngularJS是一个用来建立app的框架，而JQuery则是一个简化“HTML文档操作、事件处理、动画和Ajax”的库。这是两者最基本的区别，AngularJS致力于程序的体系结构，与HTML页面无关。

为了更好的理解如何建立一个AngularJS程序，请停止使用jQuery。JQuery使开发人员以现存的HTML标准思考问题，但正如文档里所说的，“AngularJS能够让你在应用程序中扩张HTML这个词汇”。

DOM操作应该只在指令中完成，但这并不意味着他们只能用JQuery封装。在你使用JQuery之前，你应该总是去想一下这个功能是不是AngularJS已经提供了。当指令互相依赖时能够创建强大的工具，这确实很强大。

但一个非常棒的JQuery是必需品时，这一天可能会到来，但在一开始就引入它，是一个常见的错误。

### 结论

AngularJS是一卓越的框架，在社区的帮助下始终在进步。虽说AngularJS仍然是一个不断发展的概念，但我希望人们能够遵循以上谈到的这些约定，避免开发AngularJS应用所遇到的那些问题。
