---
title: 从本质认识JavaScript的原型继承和类继承
tags: [javascript,ES6]
date: 2016/03/31
---

JavaScript发展到今天，和其他语言不一样的一个特点是，有各种各样的“继承方式”，或者稍微准确一点的说法，叫做有各种各样的基于prototype的模拟类继承实现方式。

在ES6之前，JavaScript没有类继承的概念，因此使用者为了代码复用的目的，只能参考其他语言的“继承”，然后用prototype来模拟出对应的实现，于是有了各种继承方式，比如《JavaScript高级程序设计》上说的 原型链，借用构造函数，组合继承，原型式继承，寄生式继承，寄生组合式继承 等等

那么多继承方式，让第一次接触这一块的小伙伴们内心有点崩溃。然而，之所以有那么多继承方式，其实还是因为“模拟”二字，因为我们在说继承的时候不是在研究prototype本身，而是在用prototype和JS特性来模拟别的语言的类继承。

我们现在抛开这些种类繁多的继承方式，来看一下prototype的本质和我们为什么要模拟类继承。

### 原型继承

“原型” 这个词本身源自心理学，指神话、宗教、梦境、幻想、文学中不断重复出现的意象，它源自民族记忆和原始经验的集体潜意识。

所以，原型是一种抽象，代表事物表象之下的联系，用简单的话来说，就是原型描述事物与事物之间的相似性.

想象一个小孩子如何认知这个世界：

当小孩子没见过老虎的时候，大人可能会教他，老虎啊，就像是一只大猫。如果这个孩子碰巧常常和邻居家的猫咪玩耍，那么她不用去动物园见到真实的老虎，就能想象出老虎大概是长什么样子。

这个故事有个更简单的表达，叫做“照猫画虎”。如果我们用JavaScript的原型来描述它，就是：

```
function Tiger(){
    //...
}

Tiger.prototype = new Cat(); //老虎的原型是一只猫
```

很显然，“照猫画虎”（或者反过来“照虎画猫”，也可以，取决孩子于先认识老虎还是先认识猫）是一种认知模式，它让人类儿童不需要在脑海里重新完全构建一只老虎的全部信息，而可以通过她熟悉的猫咪的“复用”得到老虎的大部分信息，接下来她只需要去到动物园，去观察老虎和猫咪的不同部分，就可以正确认知什么是老虎了。这段话用JavaScript可以描述如下：

```
function Cat(){

}
//小猫喵喵叫
Cat.prototype.say = function(){    
  return "喵";
}
//小猫会爬树
Cat.prototype.climb = function(){
  return "我会爬树";
}

function Tiger(){

}
Tiger.prototype = new Cat();

//老虎的叫声和小猫不同，但老虎也会爬树
Tiger.prototype.say = function(){
  return "嗷";
}
```

所以，原型可以通过描述两个事物之间的相似关系来复用代码，我们可以把这种复用代码的模式称为原型继承。

### 类继承

几年之后，当时的小孩子长大了，随着她的知识结构不断丰富，她认识世界的方式也发生了一些变化，她学会了太多的动物，有喵喵叫的猫，百兽之王狮子，优雅的丛林之王老虎，还有豺狼、大象等等。

这时候，单纯的相似性的认知方式已经很少被使用在如此丰富的知识内容里，更加严谨的认知方式——分类，开始被更频繁使用。

这时候当年的小孩会说，猫和狗都是动物，如果她碰巧学习的是专业的生物学，她可能还会说猫和狗都是脊索门哺乳纲，于是，相似性被“类”这一种更高程度的抽象表达取代，我们用JavaScript来描述：

```
class Animal{
    eat(){}
    say(){}
    climb(){}
    ...
}
class Cat extends Animal{
    say(){return "喵"}
}
class Dog extends Animal{
    say(){return "汪"}
}
```

### 原型继承和类继承

所以，原型继承和类继承是两种认知模式，本质上都是为了抽象（复用代码）。相对于类，原型更初级且更灵活。因此当一个系统内没有太多关联的事物的时候，用原型明显比用类更灵活便捷。

原型继承的便捷性表现在系统中对象较少的时候，原型继承不需要构造额外的抽象类和接口就可以实现复用。（如系统里只有猫和狗两种动物的话，没必要再为它们构造一个抽象的“动物类”）

原型继承的灵活性还表现在复用模式更加灵活。由于原型和类的模式不一样，所以对复用的判断标准也就不一样，例如把一个红色皮球当做一个太阳的原型，当然是可以的（反过来也行），但显然不能将“恒星类”当做太阳和红球的公共父类（倒是可以用“球体”这个类作为它们的公共父类）。

既然原型本质上是一种认知模式可以用来复用代码，那我们为什么还要模拟“类继承”呢？在这里面我们就得看看原型继承有什么问题。

### 原型继承的问题

由于我们刚才前面举例的猫和老虎的构造器没有参数，因此大家很可能没发现问题，现在我们试验一个有参数构造器的原型继承：

```
function Vector2D(x, y){
  this.x = x;
  this.y = y;
}
Vector2D.prototype.length = function(){
  return Math.sqrt(this.x * this.x + this.y * this.y);
}

function Vector3D(x, y, z){
  Vector2D.call(this, x, y);
  this.z = z;
}
Vector3D.prototype = new Vector2D();

Vector3D.prototype.length = function(){
  return Math.sqrt(this.x * this.x + this.y * this.y + this.z * this.z);
}

var p = new Vector3D(1, 2, 3);
console.log(p.x, p.y, p.z, p.length(), p instanceof Vector2D);
```

上面这段代码里面我们看到我们用 Vector2D 的实例作为 Vector3D 的原型，在 Vector3D 的构造器里面我们还可以调用 Vector2D 的构造器来初始化 x、y。

但是，如果认真研究上面的代码，会发现一个小问题，在中间描述原型继承的时候：

```
Vector3D.prototype = new Vector2D();
```

我们其实无参数地调用了一次 Vector2D 的构造器！

这一次调用是不必要的，而且，因为我们的 Vector2D 的构造器足够简单并且没有副作用，所以我们这次无谓的调用除了稍稍消耗了性能之外，并不会带来太严重的问题。

但在实际项目中，我们有些组件的构造器比较复杂，或者操作DOM，那么这种情况下无谓多调用一次构造器，显然是有可能造成严重问题的。

于是，我们得想办法克服这一次多余的构造器调用，而显然，我们发现我们可以不必要这一次多余的调用：

```
function createObjWithoutConstructor(Class){
    function T(){};
    T.prototype = Class.prototype;
    return new T();    
}
```

上面的代码中，我们通过创建一个空的构造器T，引用父类Class的prototype，然后返回new T( )，来巧妙地避开Class构造器的执行。这样，我们确实可以绕开父类构造器的调用，并将它的调用时机延迟到子类实例化的时候（本来也应该这样才合理）。

```
function Vector2D(x, y){
  this.x = x;
  this.y = y;
}
Vector2D.prototype.length = function(){
  return Math.sqrt(this.x * this.x + this.y * this.y);
}

function Vector3D(x, y, z){
  Vector2D.call(this, x, y);
  this.z = z;
}
Vector3D.prototype = createObjWithoutConstructor(Vector2D);

Vector3D.prototype.length = function(){
  return Math.sqrt(this.x * this.x + this.y * this.y + this.z * this.z);
}

var p = new Vector3D(1, 2, 3);
console.log(p.x, p.y, p.z, p.length(), p instanceof Vector2D);
```

这样，我们解决了父类构造器延迟构造的问题之后，原型继承就比较适用了，并且这样简单处理之后，使用起来还不会影响 instanceof 返回值的正确性，这是与其他模拟方式相比最大的好处。

### 模拟类继承

最后，我们利用这个原理还可以实现比较完美的类继承：

```
(function(global){"use strict"

  Function.prototype.extend = function(props){
    var Super = this; //父类构造函数

    //父类原型
    var TmpCls = function(){

    }
    TmpCls.prototype = Super.prototype;

    var superProto = new TmpCls();

    //父类构造器wrapper
    var _super = function(){
      return Super.apply(this, arguments);
    }

    var Cls = function(){
      if(props.constructor){
        //执行构造函数
        props.constructor.apply(this, arguments);
      }
      //绑定 this._super 的方法
      for(var i in Super.prototype){
        _super[i] = Super.prototype[i].bind(this);
      }
    }
    Cls.prototype = superProto;
    Cls.prototype._super = _super;

    //复制属性
    for(var i in props){
      if(i !== "constructor"){
        Cls.prototype[i] = props[i];
      }
    }  

    return Cls;
  }

  function Animal(name){
    this.name = name;
  }

  Animal.prototype.sayName = function(){
    console.log("My name is "+this.name);
  }

  var Programmer = Animal.extend({
    constructor: function(name){
      this._super(name);
    },
    sayName: function(){
      this._super.sayName(name);
    },
    program: function(){
      console.log("I\"m coding...");
    }
  });
  //测试我们的类
  var animal = new Animal("dummy"),
      akira = new Programmer("akira");
  animal.sayName();//输出 ‘My name is dummy’
  akira.sayName();//输出 ‘My name is akira’
  akira.program();//输出 ‘I"m coding...’

})(this);
```

可以比较一下ES6的类继承：

```
(function(global){"use strict"

  //类的定义
  class Animal {
    //ES6中新型构造器
      constructor(name) {
          this.name = name;
      }
      //实例方法
      sayName() {
          console.log("My name is "+this.name);
      }
  }

  //类的继承
  class Programmer extends Animal {
      constructor(name) {
        //直接调用父类构造器进行初始化
          super(name);
      }
      sayName(){
          super.sayName();
      }
      program() {
          console.log("I\"m coding...");
      }
  }
  //测试我们的类
  var animal = new Animal("dummy"),
      akira = new Programmer("akira");
  animal.sayName();//输出 ‘My name is dummy’
  akira.sayName();//输出 ‘My name is akira’
  akira.program();//输出 ‘I"m coding...’

})(this);
```

### 总结

原型继承和类继承是两种不同的认知模式，原型继承在对象不是很多的简单应用模型里比类继承更加灵活方便。然而JavaScript的原型继承在语法上有一个构造器额外调用的问题，我们只要通过 createObjWithoutConstructor 来延迟构造器的调用，就能解决这个问题。

[http://blog.h5jun.com](http://blog.h5jun.com/post/inherits.html)