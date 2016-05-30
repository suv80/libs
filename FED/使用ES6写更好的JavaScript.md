---
title: 使用 ES6 写更好的 JavaScript
tages: [javascript,es6]
date: 2016/05/30
---

## 使用 ES6 写更好的 JavaScript part I：广受欢迎新特性

### 介绍

在ES2015规范敲定并且Node.js增添了大量的函数式子集的背景下，我们终于可以拍着胸脯说：未来就在眼前。

… 我早就想这样说了

但这是真的。V8引擎将很快实现规范，而且Node已经添加了大量可用于生产环境的ES2015特性。下面要列出的是一些我认为很有必要的特性，而且这些特性是不使用需要像Babel或者Traceur这样的翻译器就可以直接使用的。

这篇文章将会讲到三个相当流行的ES2015特性，并且已经在Node中支持了了：

+ 用let和const声明块级作用域；
+ 箭头函数；
+ 简写属性和方法。

让我们马上开始。

### let和const声明块级作用域

作用域是你程序中变量可见的区域。换句话说就是一系列的规则，它们决定了你声明的变量在哪里是可以使用的。

大家应该都听过 ，在JavaScript中只有在函数内部才会创造新的作用域。然而你创建的98%的作用域事实上都是函数作用域，其实在JavaScript中有三种创建新作用域的方法。你可以这样：

1. 创建一个函数。你应该已经知道这种方式。
2. 创建一个catch块。 我绝对没哟开玩笑.
3. 创建一个代码块。如果你用的是ES2015，在一段代码块中用let或者const声明的变量会限制它们只在这个块中可见。这叫做块级作用域。

一个代码块就是你用花括号包起来的部分。 { 像这样 }。在if/else声明和try/catch/finally块中经常出现。如果你想利用块作用域的优势，你可以用花括号包裹任意的代码来创建一个代码块

考虑下面的代码片段。

```javascript
// 在 Node 中你需要使用 strict 模式尝试这个
"use strict";

var foo = "foo";
function baz() {
    if (foo) {
        var bar = "bar";
        let foobar = foo + bar;
    }
    // foo 和 bar 这里都可见 
    console.log("This situation is " + foo + bar + ". I'm going home.");

    try {
        console.log("This log statement is " + foobar + "! It threw a ReferenceError at me!");
    } catch (err) {
        console.log("You got a " + err + "; no dice.");
    }

    try {
        console.log("Just to prove to you that " + err + " doesn't exit outside of the above `catch` block.");
    } catch (err) {
        console.log("Told you so.");
    }
}

baz();

try {
    console.log(invisible);
} catch (err) {
    console.log("invisible hasn't been declared, yet, so we get a " + err);
}
let invisible = "You can't see me, yet"; // let 声明的变量在声明前是不可访问的
```

还有些要强调的

+ 注意foobar在if块之外是不可见的，因为我们没有用let声明；
+ 我们可以在任何地方使用foo ，因为我们用var定义它为全局作用域可见；
+ 我们可以在baz内部任何地方使用bar， 因为var-声明的变量是在定义的整个作用域内都可见。
+ 用let or const声明的变量不能在定义前调用。换句话说，它不会像var变量一样被编译器提升到作用域的开始处。

const 与 let 类似，但有两点不同。

1. 必须给声明为const的变量在声明时赋值。不可以先声明后赋值。
2. 不能改变const变量的值，只有在创建它时可以给它赋值。如果你试图改变它的值，会得到一个TyepError。

### let & const: Who Cares?

我们已经用var将就了二十多年了，你可能在想我们真的需要新的类型声明关键字吗？（这里作者应该是想表达这个意思）

问的好，简单的回答就是–不， 并不真正需要。但在可以用let和const的地方使用它们很有好处的。

+ let和const声明变量时都不会被提升到作用域开始的地方，这样可以使代码可读性更强，制造尽可能少的迷惑。
+ 它会尽可能的约束变量的作用域，有助于减少令人迷惑的命名冲突。
+ 这样可以让程序只有在必须重新分配变量的情况下重新分配变量。 const 可以加强常量的引用。

另一个例子就是 let 在 for 循环中的使用：

```javascript
"use strict";

var languages = ['Danish', 'Norwegian', 'Swedish'];

//会污染全局变量!
for (var i = 0; i &lt; languages.length; i += 1) {
    console.log(`${languages[i]} is a Scandinavian language.`);
}

console.log(i); // 4

for (let j = 0; j &lt; languages.length; j += 1) {
    console.log(`${languages[j]} is a Scandinavian language.`);
}

try {
    console.log(j); // Reference error
} catch (err) {
    console.log(`You got a ${err}; no dice.`);
}
```

在for循环中使用var声明的计数器并不会真正把计数器的值限制在本次循环中。 而let可以。

let在每次迭代时重新绑定循环变量有很大的优势，这样每个循环中拷贝自身 , 而不是共享全局范围内的变量。

```javascript
"use strict";

// 简洁明了
for (let i = 1; i &lt; 6; i += 1) {
    setTimeout(function() {
        console.log("I've waited " + i + " seconds!");
    }, 1000 * i);
}

// 功能完全混乱
for (var j = 0; j &lt; 6; j += 1) {
        setTimeout(function() {
        console.log("I've waited " + j + " seconds for this!");
    }, 1000 * j);
}
```

第一层循环会和你想象的一样工作。而下面的会每秒输出 “I’ve waited 6 seconds!”。

好吧，我选择狗带。

### 动态this关键字的怪异

JavaScript的this关键字因为总是不按套路出牌而臭名昭著。

事实上，它的规则相当简单。不管怎么说，this在有些情形下会导致奇怪的用法

```javascript
"use strict";

const polyglot = {
    name : "Michel Thomas",
    languages : ["Spanish", "French", "Italian", "German", "Polish"],
    introduce : function () {
        // this.name is "Michel Thomas"
        const self = this;
        this.languages.forEach(function(language) {
            // this.name is undefined, so we have to use our saved "self" variable 
            console.log("My name is " + self.name + ", and I speak " + language + ".");
        });
    }
}

polyglot.introduce();
```

在introduce里, this.name是undefined。在回调函数外面，也就是forEach中， 它指向了polyglot对象。在这种情形下我们总是希望在函数内部this和函数外部的this指向同一个对象。

问题是在JavaScript中函数会根据确定性四原则在调用时定义自己的this变量。这就是著名的动态this 机制。

这些规则中没有一个是关于查找this所描述的“附近作用域”的；也就是说并没有一个确切的方法可以让JavaScript引擎能够基于包裹作用域来定义this的含义。

这就意味着当引擎查找this的值时，可以找到值，但却和回调函数之外的不是同一个值。有两种传统的方案可以解决这个问题。

+ 在函数外面把this保存到一个变量中，通常取名self，并在内部函数中使用；
+ 或者在内部函数中调用bind阻止对this的赋值。

以上两种办法均可生效，但会产生副作用。

另一方面，如果内部函数没有设置它自己的this值，JavaScript会像查找其它变量那样查找this的值：通过遍历父作用域直到找到同名的变量。这样会让我们使用附近作用域代码中的this值，这就是著名的词法this。

如果有样的特性，我们的代码将会更加的清晰，不是吗?

### 箭头函数中的词法this

在 ES2015 中，我们有了这一特性。箭头函数不会绑定this值，允许我们利用词法绑定this关键字。这样我们就可以像这样重构上面的代码了：

```javascript
"use strict";

let polyglot = {
    name : "Michel Thomas",
    languages : ["Spanish", "French", "Italian", "German", "Polish"],
    introduce : function () {
        this.languages.forEach((language) =&gt; {
            console.log("My name is " + this.name + ", and I speak " + language + ".");
        });
    }
}
```

… 这样就会按照我们想的那样工作了。

箭头函数有一些新的语法。

```javascript
"use strict";

let languages = ["Spanish", "French", "Italian", "German", "Polish"];

// 多行箭头函数必须使用花括号， 
// 必须明确包含返回值语句
    let languages_lower = languages.map((language) =&gt; {
    return language.toLowerCase()
});

// 单行箭头函数，花括号是可省的，
// 函数默认返回最后一个表达式的值
// 你可以指明返回语句，这是可选的。
let languages_lower = languages.map((language) =&gt; language.toLowerCase());

// 如果你的箭头函数只有一个参数，可以省略括号
let languages_lower = languages.map(language =&gt; language.toLowerCase());

// 如果箭头函数有多个参数，必须用圆括号包裹
let languages_lower = languages.map((language, unused_param) =&gt; language.toLowerCase());

console.log(languages_lower); // ["spanish", "french", "italian", "german", "polish"]

// 最后，如果你的函数没有参数，你必须在箭头前加上空的括号。
(() =&gt; alert("Hello!"))();
```

MDN关于箭头函数的文档解释的很好。

### 简写属性和方法

ES2015提供了在对象上定义属性和方法的一些新方式。

### 简写方法

在 JavaScript 中， method 是对象的一个有函数值的属性：

```javascript
"use strict";

const myObject = {
    const foo = function () {
        console.log('bar');
    },
}
```

在ES2015中，我们可以这样简写：

```javascript
"use strict";

const myObject = {
    foo () {
        console.log('bar');
    },
    * range (from, to) {
        while (from &lt; to) {
            if (from === to)
                return ++from;
            else
                yield from ++;
        }
    }
}
```

注意你也可以使用生成器去定义方法。只需要在函数名前面加一个星号(*)。

这些叫做 方法定义 。和传统的函数作为属性很像，但有一些不同：

+ 只能在方法定义处调用super；
+ 不允许用new调用方法定义。

我会在随后的几篇文章中讲到super关键字。如果你等不及了， Exploring ES6中有关于它的干货。

### 简写和推导属性

ES6还引入了简写和推导属性 。

如果对象的键值和变量名是一致的，那么你可以仅用变量名来初始化你的对象，而不是定义冗余的键值对。

```javascript
"use strict";

const foo = 'foo';
const bar = 'bar';

// 旧语法
const myObject = {
    foo : foo,
    bar : bar
};

// 新语法
const myObject = { foo, bar }
```

两中语法都以foo和bar键值指向foo and bar变量。后面的方式语义上更加一致；这只是个语法糖。

当用揭示模块模式来定义一些简洁的公共 API 的定义，我常常利用简写属性的优势。

```javascript
"use strict";

function Module () {
    function foo () {
        return 'foo';
    }
    
    function bar () {
        return 'bar';
    }
    
    // 这样写:
    const publicAPI = { foo, bar }
    
    /* 不要这样写:
    const publicAPI =  {
       foo : foo,
       bar : bar
    } */ 
    
    return publicAPI;
};
```

这里我们创建并返回了一个publicAPI对象，键值foo指向foo方法，键值bar指向bar方法。

### 推导属性名

这是不常见的例子，但ES6允许你用表达式做属性名。

```javascript
"use strict";

const myObj = {
  // 设置属性名为 foo 函数的返回值
    [foo ()] () {
      return 'foo';
    }
};

function foo () {
    return 'foo';
}

console.log(myObj.foo() ); // 'foo'
```

根据Dr. Raushmayer在Exploring ES6中讲的，这种特性最主要的用途是设置属性名与Symbol值一样。

### Getter 和 Setter 方法

最后，我想提一下get和set方法，它们在ES5中就已经支持了。

```javascript
"use strict";

// 例子采用的是 MDN's 上关于 getter 的内容
//   https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get
const speakingObj = {
    // 记录 “speak” 方法调用过多少次
    words : [],
    
    speak (word) {
        this.words.push(word);
        console.log('speakingObj says ' + word + '!');
    },
    
    get called () {
        // 返回最新的单词
        const words = this.words;
        if (!words.length)
            return 'speakingObj hasn\'t spoken, yet.';
        else
            return words[words.length - 1];
    }
};

console.log(speakingObj.called); // 'speakingObj hasn't spoken, yet.'

speakingObj.speak('blargh'); // 'speakingObj says blargh!'

console.log(speakingObj.called); // 'blargh'
```

使用getters时要记得下面这些:

+ Getters不接受参数；
+ 属性名不可以和getter函数重名；
+ 可以用Object.defineProperty(OBJECT, "property name", { get : function () { . . . } }) 动态创建 getter

作为最后这点的例子，我们可以这样定义上面的 getter 方法：

```javascript
"use strict";

const speakingObj = {
    // 记录 “speak” 方法调用过多少次
    words : [],
    
    speak (word) {
        this.words.push(word);
        console.log('speakingObj says ' + word + '!');
    }
};

// 这只是为了证明观点。我是绝对不会这样写的
function called () {
    // 返回新的单词
    const words = this.words;
    if (!words.length)
        return 'speakingObj hasn\'t spoken, yet.';
    else
        return words[words.length - 1];
};

Object.defineProperty(speakingObj, "called", get : getCalled ) 
除了 getters，还有 setters。像平常一样，它们通过自定义的逻辑给对象设置属性。

"use strict";

// 创建一个新的 globetrotter（环球者）！
const globetrotter = {
    // globetrotter 现在所处国家所说的语言 
    const current_lang = undefined,
    
    // globetrotter 已近环游过的国家
    let countries = 0,
    
    // 查看环游过哪些国家了
    get countryCount () {
        return this.countries;
    }, 
    
    // 不论 globe trotter 飞到哪里，都重新设置他的语言
    set languages (language) {
        // 增加环游过的城市数
        countries += 1;
    
        // 重置当前语言
        this.current_lang = language; 
    };
};

globetrotter.language = 'Japanese';
globetrotter.countryCount(); // 1

globetrotter.language = 'Spanish';
globetrotter.countryCount(); // 2
```

上面讲的关于getters的也同样适用于setters，但有一点不同：

+ getter不接受参数，setters必须接受正好一个参数。

破坏这些规则中的任意一个都会抛出一个错误。

既然 Angular 2 正在引入TypeCript并且把class带到了台前，我希望get and set能够流行起来… 但还有点希望它们不要流行起来。

### 结论

未来的JavaScript正在变成现实，是时候把它提供的东西都用起来了。这篇文章里，我们浏览了 ES2015的三个很流行的特性：

+ let和const带来的块级作用域；
+ 箭头函数带来的this的词法作用域；
+ 简写属性和方法，以及getter和setter函数的回顾。

## 使用 ES6 编写更好的 JavaScript Part II：深入探究 [类]

### 辞旧迎新

在本文的开始，我们要说明一件事：

从本质上说，ES6的classes主要是给创建老式构造函数提供了一种更加方便的语法，并不是什么新魔法 —— Axel Rauschmayer，Exploring ES6作者

从功能上来讲，class声明就是一个语法糖，它只是比我们之前一直使用的基于原型的行为委托功能更强大一点。本文将从新语法与原型的关系入手，仔细研究ES2015的class关键字。文中将提及以下内容：

+ 定义与实例化类；
+ 使用extends创建子类；
+ 子类中super语句的调用；
+ 以及重要的标记方法（symbol method）的例子。

在此过程中，我们将特别注意 class 声明语法从本质上是如何映射到基于原型代码的。

让我们从头开始说起。

### 退一步说：Classes不是什么

JavaScript的『类』与Java、Python或者其他你可能用过的面向对象语言中的类不同。其实后者可能称作面向『类』的语言更为准确一些。

在传统的面向类的语言中，我们创建的类是对象的模板。需要一个新对象时，我们实例化这个类，这一步操作告诉语言引擎将这个类的方法和属性复制到一个新实体上，这个实体称作实例。实例是我们自己的对象，且在实例化之后与父类毫无内在联系。

而JavaScript没有这样的复制机制。在JavaScript中『实例化』一个类创建了一个新对象，但这个新对象却不独立于它的父类。

正相反，它创建了一个与原型相连接的对象。即使是在实例化之后，对于原型的修改也会传递到实例化的新对象去。

原型本身就是一个无比强大的设计模式。有许多使用了原型的技术模仿了传统类的机制，class便为这些技术提供了简洁的语法。

总而言之：

+ JavaScript不存在Java和其他面向对象语言中的类概念；
+ JavaScript 的class很大程度上只是原型继承的语法糖，与传统的类继承有很大的不同。

搞清楚这些之后，让我们先看一下class。

### 类基础：声明与表达式

我们使用class 关键字创建类，关键字之后是变量标识符，最后是一个称作类主体的代码块。这种写法称作类的声明。没有使用extends关键字的类声明被称作基类：

```javascript
"use strict";

// Food 是一个基类
class Food {

    constructor (name, protein, carbs, fat) {
        this.name = name;
        this.protein = protein;
        this.carbs = carbs;
        this.fat = fat;
    }

    toString () {
        return `${this.name} | ${this.protein}g P :: ${this.carbs}g C :: ${this.fat}g F`
    }

    print () {
        console.log( this.toString() );
    }
}

const chicken_breast = new Food('Chicken Breast', 26, 0, 3.5);

chicken_breast.print(); // 'Chicken Breast | 26g P :: 0g C :: 3.5g F'
console.log(chicken_breast.protein); // 26 (LINE A)
```

需要注意到以下事情：

+ 类只能包含方法定义，不能有数据属性；
+ 定义方法时，可以使用简写方法定义；
+ 与创建对象不同，我们不能在类主体中使用逗号分隔方法定义；
+ 我们可以在实例化对象上直接引用类的属性（如 LINE A）。

类有一个独有的特性，就是 contructor 构造方法。在构造方法中我们可以初始化对象的属性。

构造方法的定义并不是必须的。如果不写构造方法，引擎会为我们插入一个空的构造方法：

```javascript
"use strict";

class NoConstructor {
    /* JavaScript 会插入这样的代码：
     constructor () { }
    */
}

const nemo = new NoConstructor(); // 能工作，但没啥意思
```

将一个类赋值给一个变量的形式叫类表达式，这种写法可以替代上面的语法形式：

```javascript
"use strict";

// 这是一个匿名类表达式，在类主体中我们不能通过名称引用它
const Food = class {
    // 和上面一样的类定义……
}

// 这是一个命名类表达式，在类主体中我们可以通过名称引用它
const Food = class FoodClass {
    // 和上面一样的类定义……

    //  添加一个新方法，证明我们可以通过内部名称引用 FoodClass……        
    printMacronutrients () {
        console.log(`${FoodClass.name} | ${FoodClass.protein} g P :: ${FoodClass.carbs} g C :: ${FoodClass.fat} g F`)
    }
}

const chicken_breast = new Food('Chicken Breast', 26, 0, 3.5);
chicken_breast.printMacronutrients(); // 'Chicken Breast | 26g P :: 0g C :: 3.5g F'

// 但是不能在外部引用
try {
    console.log(FoodClass.protein); // 引用错误
} catch (err) {
    // pass
}
```

这一行为与匿名函数与命名函数表达式很类似。

### 使用extends创建子类以及使用super调用

使用extends创建的类被称作子类，或派生类。这一用法简单明了，我们直接在上面的例子中构建：

```javascript
"use strict";

// FatFreeFood 是一个派生类
class FatFreeFood extends Food {

    constructor (name, protein, carbs) {
        super(name, protein, carbs, 0);
    }

    print () {
        super.print();
        console.log(`Would you look at that -- ${this.name} has no fat!`);
    }

}

const fat_free_yogurt = new FatFreeFood('Greek Yogurt', 16, 12);
fat_free_yogurt.print(); // 'Greek Yogurt | 26g P :: 16g C :: 0g F  /  Would you look at that -- Greek Yogurt has no fat!'
```

派生类拥有我们上文讨论的一切有关基类的特性，另外还有如下几点新特点：

+ 子类使用class关键字声明，之后紧跟一个标识符，然后使用extend关键字，最后写一个任意表达式。这个表达式通常来讲就是个标识符，但理论上也可以是函数。
+ 如果你的派生类需要引用它的父类，可以使用super关键字。
+ 一个派生类不能有一个空的构造函数。即使这个构造函数就是调用了一下super()，你也得把它显式的写出来。但派生类却可以没有构造函数。
+ 在派生类的构造函数中，必须先调用super，才能使用this关键字（译者注：仅在构造函数中是这样，在其他方法中可以直接使用this）。

在JavaScript中仅有两个super关键字的使用场景：

1. 在子类构造函数中调用。如果初始化派生类是需要使用父类的构造函数，我们可以在子类的构造函数中调用super(parentConstructorParams)，传递任意需要的参数。
2. 引用父类的方法。在常规方法定义中，派生类可以使用点运算符来引用父类的方法：super.methodName。

我们的 FatFreeFood 演示了这两种情况：

1. 在构造函数中，我们简单的调用了super，并将脂肪的量传入为0。
2. 在我们的print方法中，我们先调用了super.print，之后才添加了其他的逻辑。

不管你信不信，我反正是信了以上说的已涵盖了有关class的基础语法，这就是你开始实验需要掌握的全部内容。

### 深入学习原型

现在我们开始关注class是怎么映射到JavaScript内部的原型机制的。我们会关注以下几点：

+ 使用构造调用创建对象；
+ 原型连接的本质；
+ 属性和方法委托；
+ 使用原型模拟类。
+ 使用构造调用创建对象

构造函数不是什么新鲜玩意儿。使用new关键字调用任意函数会使其返回一个对象 —— 这一步称作创建了一个构造调用，这种函数通常被称作构造器：

```javascript
"use strict";

function Food (name, protein, carbs, fat) {
    this.name    = name;
    this.protein = protein;
    this.carbs   = carbs;
    this.fat     = fat;
}

// 使用 'new' 关键字调用 Food 方法，就是构造调用，该操作会返回一个对象
const chicken_breast = new Food('Chicken Breast', 26, 0, 3.5);
console.log(chicken_breast.protein) // 26

// 不用 'new' 调用 Food 方法，会返回 'undefined'
const fish = Food('Halibut', 26, 0, 2);
console.log(fish); // 'undefined'
```

当我们使用new关键字调用函数时，JS内部执行了下面四个步骤：

1. 创建一个新对象（这里称它为O）；
2. 给O赋予一个连接到其他对象的链接，称为原型；
3. 将函数的this引用指向O；
4. 函数隐式返回O。

在第三步和第四步之间，引擎会执行你函数中的具体逻辑。

知道了这一点，我们就可以重写Food方法，使之不用new关键字也能工作：

```javascript
"use strict";

// 演示示例：消除对 'new' 关键字的依赖
function Food (name, protein, carbs, fat) {
    // 第一步：创建新对象
    const obj = { };

    // 第二步：链接原型——我们在下文会更加具体地探究原型的概念
    Object.setPrototypeOf(obj, Food.prototype);

    // 第三步：设置 'this' 指向我们的新对象
    // 尽然我们不能再运行的执行上下文中重置 `this`
    // 我们在使用 'obj' 取代 'this' 来模拟第三步
    obj.name    = name;
    obj.protein = protein;
    obj.carbs   = carbs;
    obj.fat     = fat;

    // 第四步：返回新创建的对象
    return obj;
}

const fish = Food('Halibut', 26, 0, 2);
console.log(fish.protein); // 26
```

四步中的三步都是简单明了的。创建一个对象、赋值属性、然后写一个return声明，这些操作对大多数开发者来说没有理解上的问题——然而这就是难倒众人的黑魔法原型。

### 直观理解原型链

在通常情况下，JavaScript中的包括函数在内的所有对象都会链接到另一个对象上，这就是原型。

如果我们访问一个对象本身没有的属性，JavaScript就会在对象的原型上检查该属性。换句话说，如果你对一个对象请求它没有的属性，它会对你说：『这个我不知道，问我的原型吧』。

在另一个对象上查找不存在属性的过程称作委托。

```javascript
"use strict";

// joe 没有 toString 方法……
const joe    = { name : 'Joe' },
    sara   = { name : 'Sara' };

Object.hasOwnProperty(joe, toString); // false
Object.hasOwnProperty(sara, toString); // false

// ……但我们还是可以调用它！
joe.toString(); // '[object Object]'，而不是引用错误！
sara.toString(); // '[object Object]'，而不是引用错误！
```

尽管我们的 toString 的输出完全没啥用，但请注意：这段代码没有引起任何的ReferenceError！这是因为尽管joe和sara没有toString的属性，但他们的原型有啊。

当我们寻找sara.toString()方法时，sara说：『我没有toString属性，找我的原型吧』。正如上文所说，JavaScript会亲切的询问Object.prototype 是否含有toString属性。由于原型上有这一属性，JS 就会把Object.prototype上的toString返回给我们程序并执行。

sara本身没有属性没关系——我们会把查找操作委托到原型上。

换言之，我们就可以访问到对象上并不存在的属性，只要其的原型上有这些属性。我们可以利用这一点将属性和方法赋值到对象的原型上，然后我们就可以调用这些属性，好像它们真的存在在那个对象上一样。

更给力的是，如果几个对象共享相同的原型——正如上面的joe和sara的例子一样——当我们给原型赋值属性之后，它们就都可以访问了，无需将这些属性单独拷贝到每一个对象上。

这就是为何大家把它称作原型继承——如果我的对象没有，但对象的原型有，那我的对象也能继承这个属性。

事实上，这里并没有发生什么『继承』。在面向类的语言里，继承指从父类复制属性到子类的行为。在JavaScript里，没发生这种复制的操作，事实上这就是原型继承与类继承相比的一个主要优势。

在我们探究原型究竟是怎么来的之前，我们先做一个简要回顾：

+ joe和sara没有『继承』一个toString的属性；
+ joe和sara实际上根本没有从Object.prototype上『继承』；
+ joe和sara是链接到了Object.prototype上；
+ joe和sara链接到了同一个Object.prototype上。
+ 如果想找到一个对象的（我们称它作O）原型，我们可以使用 Object.getPrototypeof(O)。

然后我们再强调一遍：对象没有『继承自』他们的原型。他们只是委托到原型上。

以上。

接下来让我们深入一下。

### 设置对象的原型

我们已了解到基本上每个对象（下文以O指代）都有原型（下文以P指代），然后当我们查找O上没有的属性，JavaScript引擎就会在P上寻找这个属性。

至此我们有两个问题：

+ 以上情况函数怎么玩？
+ 这些原型是从哪里来的？

名为Object的函数

在JavaScript引擎执行程序之前，它会创建一个环境让程序在内部执行，在执行环境中会创建一个函数，叫做Object, 以及一个关联对象，叫做Object.prototype。

换句话说，Object和Object.prototype在任意执行中的JavaScript程序中永远存在。

这个Object乍一看好像和其他函数没什么区别，但特别之处在于它是一个构造器——在调用它时返回一个新对象：

```javascript
"use strict";

typeof new Object(); // "object"
typeof Object();     // 这个 Object 函数的特点是不需要使用 new 关键字调用
```

这个Object.prototype对象是个……对象。正如其他对象一样，它有属性。

![https://i.imgsafe.org/ebbd5e3.png](https://i.imgsafe.org/ebbd5e3.png)

关于Object和Object.prototype你需要知道以下几点：

1. Object函数有一个叫做.prototype的属性，指向一个对象（Object.prototype）；
2. Object.prototype对象有一个叫做.constructor的属性，指向一个函数（Object）。

实际上，这个总体方案对于JavaScript中的所有函数都是适用的。当我们创建一个函数——下文称作 someFunction——这个函数就会有一个属性.prototype，指向一个叫做someFunction.prototype 的对象。

与之相反，someFunction.prototype对象会有一个叫做.contructor的属性，它的引用指回函数someFunction。

```javascript
"use strict";

function foo () {  console.log('Foo!');  }

console.log(foo.prototype); // 指向一个叫 'foo' 的对象
console.log(foo.prototype.constructor); // 指向 'foo' 函数

foo.prototype.constructor(); // 输出 'Foo!' —— 仅为证明确实有 'foo.prototype.constructor' 这么个方法且指向原函数
```

需要记住以下几个要点：

1. 所有的函数都有一个属性，叫做 .prototype，它指向这个函数的关联对象。
2. 所有函数的原型都有一个属性，叫做 .constructor，它指向这个函数本身。
3. 一个函数原型的 .constructor 并非必须指向创建这个函数原型的函数……有点绕，我们等下会深入探讨一下。

设置函数的原型有一些规则，在开始之前，我们先概括设置对象原型的三个规则：

1. 『默认』规则；
2. 使用new隐式设置原型；
3. 使用Object.create显式设置原型。

### 默认规则

考虑下这段代码：

```javascript
"use strict";

const foo = { status : 'foobar' };
```

十分简单，我们做的事儿就是创建一个叫foo的对象，然后给他一个叫status的属性。

然后JavaScript在幕后多做了点工作。当我们在字面上创建一个对象时，JavaScript将对象的原型指向Object.prototype并设置其原型的.constructor指向Object：

```javascript
"use strict";

const foo = { status : 'foobar' };

Object.getPrototypeOf(foo) === Object.prototype; // true
foo.constructor === Object; // true
```

### 使用new隐式设置原型

让我们再看下之前调整过的 Food 例子。

```javascript
"use strict";

function Food (name, protein, carbs, fat) {
    this.name    = name;
    this.protein = protein;
    this.carbs   = carbs;
    this.fat     = fat;
}
```

现在我们知道函数Food将会与一个叫做Food.prototype的对象关联。

当我们使用new关键字创建一个对象，JavaScript将会：

1. 设置这个对象的原型指向我们使用new调用的函数的.prototype属性；
2. 设置这个对象的.constructor指向我们使用new调用到的构造函数。

```javascript
const tootsie_roll = new Food('Tootsie Roll', 0, 26, 0);

Object.getPrototypeOf(tootsie_roll) === Food.prototype; // true
tootsie_roll.constructor === Food; // true
```

这就可以让我们搞出下面这样的黑魔法：

```javascript
"use strict";

Food.prototype.cook = function cook () {
    console.log(`${this.name} is cooking!`);
};

const dinner = new Food('Lamb Chops', 52, 8, 32);
dinner.cook(); // 'Lamb Chops are cooking!'
```

### 使用Object.create显式设置原型

最后我们可以使用Object.create方法手工设置对象的原型引用。

```javascript
"use strict";

const foo = {
    speak () {
        console.log('Foo!');
    }
};

const bar = Object.create(foo);

bar.speak(); // 'Foo!'
Object.getPrototypeOf(bar) === foo; // true
```

还记得使用new调用函数的时候，JavaScript在幕后干了哪四件事儿吗？Object.create就干了这三件事儿：

1. 创建一个新对象；
2. 设置它的原型引用；
3. 返回这个新对象。

[你可以自己去看下MDN上写的那个polyfill。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) 
（译者注：polyfill就是给老代码实现现有新功能的补丁代码，这里就是指老版本JS没有Object.create函数，MDN上有手工撸的一个替代方案）

### 模拟 class 行为

直接使用原型来模拟面向类的行为需要一些技巧。

```javascript
"use strict";

function Food (name, protein, carbs, fat) {
    this.name    = name;
    this.protein = protein;
    this.carbs   = carbs;
    this.fat     = fat;
}

Food.prototype.toString = function () {
    return `${this.name} | ${this.protein}g P :: ${this.carbs}g C :: ${this.fat}g F`;
};

function FatFreeFood (name, protein, carbs) {
    Food.call(this, name, protein, carbs, 0);
}

// 设置 "subclass" 关系
// =====================
// LINE A :: 使用 Object.create 手动设置 FatFreeFood's 『父类』.
FatFreeFood.prototype = Object.create(Food.prototype);

// LINE B :: 手工重置 constructor 的引用
Object.defineProperty(FatFreeFood.constructor, "constructor", {
    enumerable : false,
    writeable  : true,
    value      : FatFreeFood
});
```

在Line A，我们需要设置FatFreeFood.prototype使之等于一个新对象，这个新对象的原型引用是Food.prototype。如果没这么搞，我们的子类就不能访问『超类』的方法。

不幸的是，这个导致了相当诡异的结果：FatFreeFood.constructor是Function，而不是FatFreeFood。为了保证一切正常，我们需要在Line B手工设置FatFreeFood.constructor。

让开发者从使用原型对类行为笨拙的模仿中脱离苦海是class关键字的产生动机之一。它确实也提供了避免原型语法常见陷阱的解决方案。

现在我们已经探究了太多关于JavaScript的原型机制，你应该更容易理解class关键字让一切变得多么简单了吧！

### 深入探究下方法

现在我们已了解到JavaScript原型系统的必要性，我们将深入探究一下类支持的三种方法，以及一种特殊情况，以结束本文的讨论。

+ 构造器；
+ 静态方法；
+ 原型方法；
+ 一种原型方法的特殊情况：『标记方法』。

并非我提出的这三组方法，这要归功于Rauschmayer博士在探索ES6一书中的定义。

### 类构造器

一个类的constructor方法用于关注我们的初始化逻辑，constructor方法有以下几个特殊点：

1. 只有在构造方法里，我们才可以调用父类的构造器；
2. 它在背后处理了所有设置原型链的工作；
3. 它被用作类的定义。

第二点就是在JavaScript中使用class的一个主要好处，我们来引用一下《探索 ES6》书里的15.2.3.1 的标题：

> 子类的原型就是超类

正如我们所见，手工设置非常繁琐且容易出错。如果我们使用class关键字，JavaScript在内部会负责搞定这些设置，这一点也是使用class的优势。

第三点有点意思。在JavaScript中类仅仅是个函数——它等同于与类中的constructor方法。

```javascript
"use strict";

class Food {
    // 和之前一样的类定义……
}

typeof Food; // 'function'
```

与一般把函数作为构造器的方式不同，我们不能不用new关键字而直接调用类构造器：

```javascript
const burrito = Food('Heaven', 100, 100, 25); // 类型错误
```

这就引发了另一个问题：当我们不用new调用函数构造器的时候发生了什么？

简短的回答是：对于任何没有显式返回的函数来说都是返回undefined。我们只需要相信用我们构造函数的用户都会使用构造调用。这就是社区为何约定构造方法的首字母大写：提醒使用者要用new来调用。

```javascript
"use strict";

function Food (name, protein, carbs, fat) {
    this.name    = name;
    this.protein = protein;
    this.carbs   = carbs;
    this.fat     = fat;
}

const fish = Food('Halibut', 26, 0, 2); // D'oh . . .
console.log(fish); // 'undefined'
```

长一点的回答是：返回undefined，除非你手工检测是否使用被new调用，然后进行自己的处理。

ES2015引入了一个属性使得这种检测变得简单: [new.target](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new.target).

new.target是一个定义在所有使用new调用的函数上的属性，包括类构造器。 当我们使用new关键字调用函数时，函数体内的new.target的值就是这个函数本身。如果函数没有被new调用，这个值就是undefined。

```javascript
"use strict";

// 强行构造调用
function Food (name, protein, carbs, fat) {
    // 如果用户忘了手工调用一下
    if (!new.target)
        return new Food(name, protein, carbs, fat);

    this.name    = name;
    this.protein = protein;
    this.carbs   = carbs;
    this.fat     = fat;
}

const fish = Food('Halibut', 26, 0, 2); // 糟了，不过没关系！
fish; // 'Food {name: "Halibut", protein: 20, carbs: 5, fat: 0}'
```

在ES5里用起来也还行：

```javascript
"use strict";

function Food (name, protein, carbs, fat) {

    if (!(this instanceof Food))
        return new Food(name, protein, carbs, fat);

    this.name    = name;
    this.protein = protein;
    this.carbs   = carbs;
    this.fat     = fat;
}
```

MDN文档讲述了new.target的更多细节，而且给有兴趣者配上了ES2015规范作为参考。规范里有关 [[Construct]] 的描述很有启发性。

### 静态方法

静态方法是构造方法自己的方法，不能被类的实例化对象调用。我们使用static关键字定义静态方法。

```javascript
"use strict";

class Food {
     // 和之前一样……

     // 添加静态方法
     static describe () {
         console.log('"Food" 是一种存储了营养信息的数据类型');
     }
}

Food.describe(); // '"Food" 是一种存储了营养信息的数据类型'
```

静态方法与老式构造函数中直接属性赋值相似：

```javascript
"use strict";

function Food (name, protein, carbs, fat) {
    Food.count += 1;

    this.name    = name;
    this.protein = protein;
    this.carbs   = carbs;
    this.fat     = fat;
}

Food.count = 0;
Food.describe = function count () {
    console.log(`你创建了 ${Food.count} 个 food`);
};

const dummy = new Food();
Food.describe(); // "你创建了 1 个 food"
```

### 原型方法

任何不是构造方法和静态方法的方法都是原型方法。之所以叫原型方法，是因为我们之前通过给构造函数的原型上附加方法的方式来实现这一功能。

```javascript
"use strict";

// 使用 ES6：
class Food {

    constructor (name, protein, carbs, fat) {
        this.name = name;
        this.protein = protein;
        this.carbs = carbs;
        this.fat = fat;
    }

    toString () {  
        return `${this.name} | ${this.protein}g P :: ${this.carbs}g C :: ${this.fat}g F`;
    }

    print () {  
        console.log( this.toString() );  
    }
}

// 在 ES5 里：
function Food  (name, protein, carbs, fat) {
    this.name = name;
    this.protein = protein;
    this.carbs = carbs;
    this.fat = fat;
}

// 『原型方法』的命名大概来自我们之前通过给构造函数的原型上附加方法的方式来实现这一功能。
Food.prototype.toString = function toString () {
    return `${this.name} | ${this.protein}g P :: ${this.carbs}g C :: ${this.fat}g F`;
};

Food.prototype.print = function print () {
    console.log( this.toString() );
};
```

应该说明，在方法定义时完全可以使用生成器。

```javascript
"use strict";

class Range {

    constructor(from, to) {
        this.from = from;
        this.to   = to;
    }

    * generate () {
        let counter = this.from,
            to      = this.to;

        while (counter &lt; to) {
            if (counter == to)
                return counter++;
            else
                yield counter++;
        }
    }
}

const range = new Range(0, 3);
const gen = range.generate();
for (let val of range.generate()) {
    console.log(`Generator 的值是 ${ val }. `);
    //  Prints:
    //    Generator 的值是 0.
    //    Generator 的值是 1.
    //    Generator 的值是 2.
}
```

### 标志方法

最后我们说说标志方法。这是一些名为Symbol值的方法，当我们在自定义对象中使用内置构造器时，JavaScript引擎可以识别并使用这些方法。

MDN文档提供了一个Symbol是什么的简要概览：

Symbol是一个唯一且不变的数据类型，可以作为一个对象的属性标示符。

创建一个新的symbol，会给我们提供一个被认为是程序里的唯一标识的值。这一点对于命名对象的属性十分有用：我们可以确保不会不小心覆盖任何属性。使用Symbol做键值也不是无数的，所以他们很大程度上对外界是不可见的（也不完全是，可以通过Reflect.ownKeys获得）

```javascript
"use strict";

const secureObject = {
    // 这个键可以看作是唯一的
    [new Symbol("name")] : 'Dr. Secure A. F.'
};

console.log( Object.getKeys(superSecureObject) ); // [] -- 标志属性不太好获取    
console.log( Reflect.ownKeys(secureObject) ); // [Symbol("name")] -- 但也不是完全隐藏的
```

对我们来讲更有意思的是，这给我们提供了一种方式来告诉 JavaScript 引擎使用特定方法来达到特定的目的。

所谓的『众所周知的Symbol』是一些特定对象的键，当你在定义对象中使用时他们时，JavaScript引擎会触发一些特定方法。

这对于JavaScript来说有点怪异，我们还是看个例子吧：

```javascript
"use strict";

// 继承 Array 可以让我们直观的使用 'length'
// 同时可以让我们访问到内置方法，如
// map、filter、reduce、push、pop 等
class FoodSet extends Array {

    // foods 把传递的任意参数收集为一个数组
    // 参见：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator
    constructor(...foods) {
        super();
        this.foods = [];
        foods.forEach((food) =&gt; this.foods.push(food))
    }

     // 自定义迭代器行为，请注意，这不是多么好用的迭代器，但是个不错的例子
     // 键名前必须写星号
     * [Symbol.iterator] () {
        let position = 0;
        while (position &lt; this.foods.length) {
          if (position === this.foods.length) {
              return "Done!"
          } else {
              yield `${this.foods[ position++ ]} is the food item at position ${position}`;
          }
         }
     }

     // 当我们的用户使用内置的数组方法，返回一个数组类型对象
     // 而不是 FoodSet 类型的。这使得我们的 FoodSet 可以被一些
     // 期望操作数组的代码操作
     static get [Symbol.species] () {
         return Array;
     }
}

const foodset = new FoodSet(new Food('Fish', 26, 0, 16), new Food('Hamburger', 26, 48, 24));

// 当我们使用 for ... of 操作 FoodSet 时，JavaScript 将会使用
// 我们之前用 [Symbol.iterator] 做键值的方法
for (let food of foodset) {
    // 打印全部 food
    console.log( food );
}

// 当我们执行数组的 `filter` 方法时，JavaScript 创建并返回一个新对象
// 我们在什么对象上执行 `filter` 方法，新对象就使用这个对象作为默认构造器来创建
// 然而大部分代码都希望 filter 返回一个数组，于是我们通过重写 [Symbol.species]
// 的方式告诉 JavaScript 使用数组的构造器
const healthy_foods = foodset.filter((food) =&gt; food.name !== 'Hamburger');

console.log( healthy_foods instanceof FoodSet ); //
console.log( healthy_foods instanceof Array );
```

当你使用for...of遍历一个对象时，JavaScript将会尝试执行对象的迭代器方法，这一方法就是该对象 Symbol.iterator属性上关联的方法。如果我们提供了自己的方法定义，JavaScript就会使用我们自定义的。如果没有自己制定的话，如果有默认的实现就用默认的，没有的话就不执行。

Symbo.species更奇异了。在自定义的类中，默认的Symbol.species函数就是类的构造函数。当我们的子类有内置的集合（例如Array和Set）时，我们通常希望在使用父类的实例时也能使用子类。

通过方法返回父类的实例而不是派生类的实例，使我们更能确保我们子类在大多数代码里的可用性。而Symbol.species可以实现这一功能。

如果不怎么需要这个功能就别费力去搞了。Symbol的这种用法——或者说有关Symbol的全部用法——都还比较罕见。这些例子只是为了演示：

1. 我们可以在自定义类中使用JavaScript内置的特定构造器；
2. 用两个普通的例子展示了怎么实现这一点。

### 结论

ES2015的class关键字没有带给我们 Java 里或是SmallTalk里那种『真正的类』。宁可说它只是提供了一种更加方便的语法来创建通过原型关联的对象，本质上没有什么新东西。

## 使用ES6写更好的JavaScript part III：好用的集合和反引号

### 简介

ES2015发生了一些重大变革，像promises和generators. 但并非新标准的一切都高不可攀。 – 相当一部分新特性可以快速上手。

在这篇文章里，我们来看下新特性带来的好处:

+ 新的集合: map，weakmap，set， weakset
+ 大部分的new String methods
+ 模板字符串。

我们开始这个系列的最后一章吧。

### 模板字符串

模板字符串 解决了三个痛点，允许你做如下操作:

1. 定义在字符串内部的表达式，称为 字符串插值。
2. 写多行字符串无须用换行符 (\n) 拼接。
3. 使用“raw”字符串 – 在反斜杠内的字符串不会被转义，视为常量。

```javascript
“use strict”;

/* 三个模板字符串的例子:

字符串插值，多行字符串，raw 字符串。
================================= */
// ================================== 
// 1. 字符串插值 :: 解析任何一个字符串中的表达式。 
console.log(1 + 1 = ${1 + 1});

// ================================== 
// 2. 多行字符串 :: 这样写: 
let childe_roland = 
I saw them and I knew them all. And yet <br> Dauntless the slug-horn to my lips I set, <br> And blew “Childe Roland to the Dark Tower came.”

// … 代替下面的写法: 
child_roland = 
‘I saw them and I knew them all. And yet\n’ + 
‘Dauntless the slug-horn to my lips I set,\n’ + 
‘And blew “Childe Roland to the Dark Tower came.”’;

// ================================== 
// 3. raw 字符串 :: 在字符串前加 raw 前缀，javascript 会忽略转义字符。 
// 依然会解析包在 ${} 的表达式 
const unescaped = String.rawThis ${string()} doesn't contain a newline!\n

function string () { return “string”; }

console.log(unescaped); // ‘This string doesn’t contain a newline!\n’ – 注意 \n 会被原样输出

// 你可以像 React 使用 JSX 一样，用模板字符串创建 HTML 模板 
const template = 
`

Example

I’m a pure JS & HTML template!

`

function getClass () { 
// Check application state, calculate a class based on that state 
return “some-stateful-class”; 
}

console.log(template); // 这样使用略显笨，自己试试吧！

// 另一个常用的例子是打印变量名: 
const user = { name : ‘Joe’ };

console.log(“User’s name is ” + user.name + “.”); // 有点冗长 
console.log(User's name is ${user.name}.); // 这样稍好一些
```

1. 使用字符串插值，用反引号代替引号包裹字符串，并把我们想要的表达式嵌入在${}中。

2. 对于多行字符串，只需要把你要写的字符串包裹在反引号里，在要换行的地方直接换行。 JavaScript 会在换行处插入新行。
3. 使用原生字符串，在模板字符串前加前缀String.raw，仍然使用反引号包裹字符串。

模板字符串或许只不过是一种语法糖 … 但它比语法糖略胜一筹。

### 新的字符串方法

ES2015也给String新增了一些方法。他们主要归为两类:

1. 通用的便捷方法
2. 扩充 Unicode 支持的方法。

在本文里我们只讲第一类，同时unicode特定方法也有相当好的用例 。如果你感兴趣的话，这是地址在MDN的文档里，有一个关于字符串新方法的完整列表。

### startsWith & endsWith

对新手而言，我们有String.prototype.startsWith。 它对任何字符串都有效，它需要两个参数:

1. 一个是 search string 还有
2. 整形的位置参数 n。这是可选的。

String.prototype.startsWith方法会检查以nth位起的字符串是否以search string开始。如果没有位置参数，则默认从头开始。

如果字符串以要搜索的字符串开头返回 true，否则返回 false。

```javascript
"use strict";

const contrived_example = "This is one impressively contrived example!";

// 这个字符串是以 "This is one" 开头吗?
console.log(contrived_example.startsWith("This is one")); // true

// 这个字符串的第四个字符以 "is" 开头?
console.log(contrived_example.startsWith("is", 4)); // false

// 这个字符串的第五个字符以 "is" 开始?
console.log(contrived_example.startsWith("is", 5)); // true
```

### endsWith

String.prototype.endsWith和startswith相似: 它也需要两个参数：一个是要搜索的字符串，一个是位置。

然而String.prototype.endsWith位置参数会告诉函数要搜索的字符串在原始字符串中被当做结尾处理。

换句话说，它会切掉nth后的所有字符串，并检查是否以要搜索的字符结尾。

```javascript
"use strict";

const contrived_example = "This is one impressively contrived example!";

console.log(contrived_example.endsWith("contrived example!")); // true

console.log(contrived_example.slice(0, 11)); // "This is one"
console.log(contrived_example.endsWith("one", 11)); // true

// 通常情况下，传一个位置参数向下面这样:
function substringEndsWith (string, search_string, position) {
    // Chop off the end of the string
    const substring = string.slice(0, position);

    // 检查被截取的字符串是否已 search_string 结尾
    return substring.endsWith(search_string);
}
```

### includes

ES2015也添加了String.prototype.includes。 你需要用字符串调用它，并且要传递一个搜索项。如果字符串包含搜索项会返回true，反之返回false。

```javascript
"use strict";

const contrived_example = "This is one impressively contrived example!";

// 这个字符串是否包含单词 impressively ?
contrived_example.includes("impressively"); // true
```

ES2015之前，我们只能这样:

```javascript
"use strict";
contrived_example.indexOf("impressively") !== -1 // true
```

不算太坏。但是，String.prototype.includes是 一个改善，它屏蔽了任意整数返回值为true的漏洞。

### repeat

还有String.prototype.repeat。可以对任意字符串使用，像includes一样，它会或多或少地完成函数名指示的工作。

它只需要一个参数: 一个整型的count。使用案例说明一切，上代码:

```javascript
const na = "na";

console.log(na.repeat(5) + ", Batman!"); // 'nanananana, Batman!'
```

### raw

最后，我们有String.raw，我们在上面简单介绍过。

一个模板字符串以 String.raw 为前缀，它将不会在字符串中转义:

```javascript
/* 单右斜线要转义，我们需要双右斜线才能打印一个右斜线，\n 在普通字符串里会被解析为换行
  *   */
console.log('This string \\ has fewer \\ backslashes \\ and \n breaks the line.');

// 不想这样写的话用 raw 字符串
String.raw`This string \\ has too many \\ backslashes \\ and \n doesn't break the line.`
```

### Unicode方法

虽然我们不涉及剩余的 string 方法，但是如果我不告诉你去这个主题的必读部分就会显得我疏忽。

* Dr Rauschmayer对于Unicode in JavaScript的介绍 
* 他关于ES2015’s Unicode Support in Exploring ES6和The Absolute Minimum Every Software Developer Needs to Know About Unicode 的讨论。

无论如何我不得不跳过它的最后一部分。虽然有些老但是还是有优点的。

这里是文档中缺失的字符串方法，这样你会知道缺哪些东西了。

+ String.fromCodePoint & String.prototype.codePointAt;
+ String.prototype.normalize;
+ Unicode point escapes.

### 集合

ES2015新增了一些集合类型:

1. Map和WeakMap
2. Set和WeakSet。

合适的Map和Set类型十分方便使用，还有弱变量是一个令人兴奋的改动，虽然它对Javascript来说像舶来品一样。

### Map

map就是简单的键值对。最简单的理解方式就是和object类似，一个键对应一个值。

```javascript
"use strict";

// 我们可以把 foo 当键，bar 当值
const obj = { foo : 'bar' };

// 对象键为 foo 的值为 bar
obj.foo === 'bar'; // true
```

新的Map类型在概念上是相似的，但是可以使用任意的数据类型作为键 – 不止strings和symbols–还有除了pitfalls associated with trying to use an objects a map的一些东西。

下面的片段例举了 Map 的 API.

```javascript
"use strict";

// 构造器
let scotch_inventory = new Map();

// BASIC API METHODS
// Map.prototype.set (K, V) :: 创建一个键 K，并设置它的值为 V。
scotch_inventory.set('Lagavulin 18', 2);
scotch_inventory.set('The Dalmore', 1);

// 你可以创建一个 map 里面包含一个有两个元素的数组
scotch_inventory = new Map([['Lagavulin 18', 2], ['The Dalmore', 1]]);

// 所有的 map 都有 size 属性，这个属性会告诉你 map 里有多少个键值对。
// 用 Map 或 Set 的时候，一定要使用 size ，不能使用 length
console.log(scotch_inventory.size); // 2

// Map.prototype.get(K) :: 返回键相关的值。如果键不存在返回 undefined
console.log(scotch_inventory.get('The Dalmore')); // 1
console.log(scotch_inventory.get('Glenfiddich 18')); // undefined

// Map.prototype.has(K) :: 如果 map 里包含键 K 返回true，否则返回 false
console.log(scotch_inventory.has('The Dalmore')); // true
console.log(scotch_inventory.has('Glenfiddich 18')); // false

// Map.prototype.delete(K) :: 从 map 里删除键 K。成功返回true，不存在返回 false
console.log(scotch_inventory.delete('The Dalmore')); // true -- breaks my heart

// Map.prototype.clear() :: 清楚 map 中的所有键值对
scotch_inventory.clear();
console.log( scotch_inventory ); // Map {} -- long night

// 遍历方法
// Map 提供了多种方法遍历键值。 
//  重置值，继续探索
scotch_inventory.set('Lagavulin 18', 1);
scotch_inventory.set('Glenfiddich 18', 1);

/* Map.prototype.forEach(callback[, thisArg]) :: 对 map 里的每个键值对执行一个回调函数 
  *   你可以在回调函数内部设置 'this' 的值，通过传递一个 thisArg 参数，那是可选的而且没有太大必要那样做
  *   最后，注意回调函数已经被传了键和值 */
scotch_inventory.forEach(function (quantity, scotch) {
    console.log(`Excuse me while I sip this ${scotch}.`);
});

// Map.prototype.keys() :: 返回一个 map 中的所有键
const scotch_names = scotch_inventory.keys();
for (let name of scotch_names) {
    console.log(`We've got ${name} in the cellar.`);
}

// Map.prototype.values() :: 返回 map 中的所有值
const quantities = scotch_inventory.values();
for (let quantity of quantities) {
    console.log(`I just drank ${quantity} of . . . Uh . . . I forget`);
}

// Map.prototype.entries() :: 返回 map 的所有键值对，提供一个包含两个元素的数组 
//   以后会经常看到 map 里的键值对和 "entries" 关联 
const entries = scotch_inventory.entries();
for (let entry of entries) {
    console.log(`I remember! I drank ${entry[1]} bottle of ${entry[0]}!`);
}
```

但是Object在保存键值对的时候仍然有用。 如果符合下面的全部条件，你可能还是想用Object:

1. 当你写代码的时候，你知道你的键值对。
2. 你知道你可能不会去增加或删除你的键值对。
3. 你使用的键全都是 string 或 symbol。

另一方面，如果符合以下任意条件，你可能会想使用一个 map。

1. 你需要遍历整个map – 然而这对 object 来说是难以置信的.
2. 当你写代码的时候不需要知道键的名字或数量。
3. 你需要复杂的键，像 Object 或 别的 Map (!).

像遍历一个map一样遍历一个object是可行的，但奇妙的是–还会有一些坑潜伏在暗处。 Map更容易使用，并且增加了一些可集成的优势。然而object是以随机顺序遍历的，map是以插入的顺序遍历的。

添加随意动态键名的键值对给一个object是可行的。但奇妙的是: 比如说如果你曾经遍历过一个伪 map，你需要记住手动更新条目数。

最后一条，如果你要设置的键名不是string或symbol，你除了选择Map别无选择。

上面的这些只是一些指导性的意见，并不是最好的规则。

### WeakMap

你可能听说过一个特别棒的特性垃圾回收器，它会定期地检查不再使用的对象并清除。

To quote Dr Rauschmayer:

> WeakMap 不会阻止它的键值被垃圾回收。那意味着你可以把数据和对象关联起来不用担心内存泄漏。

换句换说，就是你的程序丢掉了WeakMap键的所有外部引用，他能自动垃圾回收他们的值。

尽管大大简化了用例，考虑到SPA(单页面应用) 就是用来展示用户希望展示的东西，像一些物品描述和一张图片，我们可以理解为API返回的JSON。

理论上来说我们可以通过缓存响应结果来减少请求服务器的次数。我们可以这样用Map :

```javascript
"use strict";

const cache = new Map();

function put (element, result) {
    cache.set(element, result);
}

function retrieve (element) {
    return cache.get(element);
}
```

… 这是行得通的，但是有内存泄漏的危险。

因为这是一个SPA，用户或许想离开这个视图，这样的话我们的 “视图”object就会失效，会被垃圾回收。

不幸的是，如果你使用的是正常的Map ,当这些object不使用时，你必须自行清除。

使用WeakMap替代就可以解决上面的问题:

```javascript
"use strict";

const cache = new WeakMap(); // 不会再有内存泄露了

// 剩下的都一样
```

这样当应用失去不需要的元素的引用时，垃圾回收系统可以自动重用那些元素。

WeakMap的API和Map相似，但有如下几点不同:

1. 在WeakMap里你可以使用object作为键。 这意味着不能以String和Symbol做键。
2. WeakMap只有set，get，has，和delete方法 – 那意味着你不能遍历weak map.
3. WeakMaps没有size属性。

不能遍历或检查WeakMap的长度的原因是，在遍历过程中可能会遇到垃圾回收系统的运行: 这一瞬间是满的，下一秒就没了。

这种不可预测的行为需要谨慎对待，TC39(ECMA第39届技术委员会)曾试图避免禁止WeakMap的遍历和长度检测。

其他的案例，可以在这里找到Use Cases for WeakMap，来自Exploring ES6.

### Set

Set就是只包含一个值的集合。换句换说，每个set的元素只会出现一次。

这是一个有用的数据类型，如果你要追踪唯一并且固定的object ,比如说聊天室的当前用户。

Set和Map有完全相同的API。主要的不同是Set没有set方法，因为它不能存储键值对。剩下的几乎相同。

```javascript
"use strict";

// 构造器
let scotch_collection = new Set();

// 基本的 API 方法
// Set.prototype.add (O) :: 和 set 一样，添加一个对象
scotch_collection.add('Lagavulin 18');
scotch_collection.add('The Dalmore');

// 你也可以用数组构造一个 set
scotch_collection = new Set(['Lagavulin 18', 'The Dalmore']);

// 所有的 set 都有一个 length 属性。这个属性会告诉你 set 里有多少对象
//   用 set 或 map 的时候，一定记住用 size，不用 length
console.log(scotch_collection.size); // 2

// Set.prototype.has(O) :: 包含对象 O 返回 true 否则返回 false
console.log(scotch_collection.has('The Dalmore')); // true
console.log(scotch_collection.has('Glenfiddich 18')); // false

// Set.prototype.delete(O) :: 删除 set 中的 O 对象，成功返回 true，不存在返回 false
scotch_collection.delete('The Dalmore'); // true -- break my heart

// Set.prototype.clear() :: 删除 set 中的所有对象
scotch_collection.clear();
console.log( scotch_collection ); // Set {} -- long night.

/* 迭代方法
 * Set 提供了多种方法遍历
 *  重新设置值，继续探索 */
scotch_collection.add('Lagavulin 18');
scotch_collection.add('Glenfiddich 18');

/* Set.prototype.forEach(callback[, thisArg]) :: 执行一个函数，回调函数
 *  set 里在每个的键值对。 You can set the value of 'this' inside 
 *  the callback by passing a thisArg, but that's optional and seldom necessary. */
scotch_collection.forEach(function (scotch) {
    console.log(`Excuse me while I sip this ${scotch}.`);
});

// Set.prototype.values() :: 返回 set 中的所有值
let scotch_names = scotch_collection.values();
for (let name of scotch_names) {
    console.log(`I just drank ${name} . . . I think.`);
}

// Set.prototype.keys() ::  对 set 来说，和 Set.prototype.values() 方法一致
scotch_names = scotch_collection.keys();
for (let name of scotch_names) {
    console.log(`I just drank ${name} . . . I think.`);
}

/* Set.prototype.entries() :: 返回 map 的所有键值对，提供一个包含两个元素的数组 
 *   这有点多余，但是这种方法可以保留 map API 的可操作性
 *    */
const entries = scotch_collection.entries();
for (let entry of entries) {
    console.log(`I got some ${entry[0]} in my cup and more ${entry[1]} in my flask!`);
}
```

### WeakSet

WeakSet相对于Set就像WeakMap相对于 Map :

1. 在WeakSet里object的引用是弱类型的。
2. WeakSet没有property属性。
3. 不能遍历WeakSet。

Weak set的用例并不多，但是这儿有一些Domenic Denicola称呼它们为“perfect for branding” – 意思就是标记一个对象以满足其他需求。

这儿是他给的例子:

```javascript
/* 下面这个例子来自 Weakset 使用案例的邮件讨论 
  *    邮件的内容和讨论的其余部分在这儿:
  *      https://mail.mozilla.org/pipermail/es-discuss/2015-June/043027.html
  */

const foos = new WeakSet();

class Foo {
  constructor() {
    foos.add(this);
  }

  method() {
    if (!foos.has(this)) {
      throw new TypeError("Foo.prototype.method called on an incompatible object!");
    }
  }
}
```

这是一个轻量科学的方法防止大家在一个没有被Foo构造出的object上使用method。

使用的WeakSet的优势是允许foo里的object使用完后被垃圾回收。

### 总结

这篇文章里，我们已经了解了ES2015带来的一些好处，从string的便捷方法和模板变量到适当的Map和Set实现。

String方法和模板字符串易于上手。同时你很快也就不用到处用weak set了，我认为你很快就会喜欢上Set和Map。

整理转载：[https://github.com/xitu/gold-miner](https://github.com/xitu/gold-miner)