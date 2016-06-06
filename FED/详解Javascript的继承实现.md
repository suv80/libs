---
title: 详解Javascript的继承实现
tags: [javascript]
date: 2015/01/28
---

我最早掌握的在js中实现继承的方法是在w3school学到的混合原型链和对象冒充的方法，在工作中，只要用到继承的时候，我都是用这个方法实现。它的实现简单，思路清晰：用对象冒充继承父类构造函数的属性，用原型链继承父类prototype 对象的方法，满足我遇到过的所有继承的场景。正因如此，我从没想过下次写继承的时候，我要换一种方式来写，直到今天晚上看了三生石上关于javascript继承系列的博客（出的很早，现在才看，真有点可惜），才发现在js里面，继承机制也可以写的如此贴近java这种后端语言的实现，确实很妙！所以我想在充分理解他博客的思路下，实现一个自己今后用得到的一个继承库。

### 1. 混合方式实现及问题

了解问题之前，先看看它的具体实现：

```
//父类构造函数
function Employee(name, salary) {
    //实例属性：姓名
    this.name = name;
    //实例属性：薪资
    this.salary = salary;
}

//通过字面量对象设置父类的原型，给父类添加实例方法
Employee.prototype = {
    //由于此处添加实例方法时也是通过修改父类原型处理的，
    //所以必须修改父类原型的constructor指向，避免父类实例的constructor属性指向Object函数
    constructor: Employee,
    getName: function () {
        return this.name;
    },
    getSalary: function () {
        return this.salary;
    },
    toString: function () {
        return this.name + ''s salary is ' + this.getSalary() + '.';
    }
}

//子类构造函数
function Manager(name, salary, percentage) {
    //对象冒充，实现属性继承（name, salary）
    Employee.apply(this, [name, salary]);
    //实例属性：提成
    this.percentage = percentage;
}

//将父类的一个实例设置为子类的原型，实现方法继承
Manager.prototype = new Employee();
//修改子类原型的constructor指向，避免子类实例的constructor属性指向父类的构造函数
Manager.prototype.constructor = Manager;
//给子类添加新的实例方法
Manager.prototype.getSalary = function () {
    return this.salary + this.salary * this.percentage;
}

var e = new Employee('jason', 5000);
var m = new Manager('tom', 8000, 0.15);

console.log(e.toString()); //jason's salary is 5000.
console.log(m.toString()); //tom's salary is 9200.

console.log(m instanceof Manager); //true
console.log(m instanceof Employee); //true
console.log(e instanceof Employee); //true
console.log(e instanceof Manager); //false
```

从结果上来说，这种继承实现方式没有问题，Manager的实例同时继承到了Employee类的实例属性和实例方法，并且通过instanceOf运算的结果也都正确。但是从代码组织和实现细节层面，这种方法还有以下几个问题：

1）代码组织不够优雅，继承实现的关键部分的逻辑是通用的，都是如下结构：

```
//将父类的一个实例设置为子类的原型，实现方法继承
SubClass.prototype = new SuperClass();
//修改子类原型的constructor指向，避免子类实例的constructor属性指向父类的构造函数
SubClass.prototype.constructor = SubClass;
//给子类添加新的实例方法
SubClass.prototype.method1 = function() {
}
SubClass.prototype.method2 = function() {
}
SubClass.prototype.method3 = function() {
}
```

这段代码缺乏封装。另外在添加子类的实例方法时，不能通过SubClass.prototype = { method1: function() {} }这种方式去设置，否则就把子类的原型整个又修改了，继承就无法实现了，这样每次都得按SubClass.prototype.method1 = function() {} 的结构去写，代码看起来很不连续。

**解决方式：**利用模块化的方式，将通用的逻辑封装起来，对外提供简单的接口，只要按照约定的接口调用，就能够简化类的构建与类的继承。具体实现请看后面的内容介绍，暂时只能提供理论的说明。

2）在给子类的原型设置成父类的实例时，调用的是new SuperClass()，这是对父类构造函数的无参调用，那么就要求父类必须有无参的构造函数。可是在javascript中，函数无法重载，所以父类不可能提供多个构造函数，在实际业务中，大部分场景下父类构造函数又不可能没有参数，为了在唯一的一个构造函数中模拟函数重载，只能借助判断arguments.length来处理。问题就是，有时候很难保证每次写父类构造函数的时候都会添加arguments.length的判断逻辑。这样的话，这个处理方式就是有风险的。要是能把构造函数里的逻辑抽离出来，让类的构造函数全部是无参函数的话，这个问题就很好解决了。

**解决方式：**把父类跟子类的构造函数全部无参化，并且在构造函数内不写任何逻辑，把构造函数的逻辑都迁移到init这个实例方法，比如前面给出的Employee和Manager的例子就能改造成下面这个样子：

```
//无参无逻辑的父类构造函数
function Employee() {}

Employee.prototype = {
    constructor: Employee,
    //把构造逻辑搬到init方法中来
    init: function (name, salary) {
            this.name = name;
            this.salary = salary;
        },
        getName: function () {
            return this.name;
        },
        getSalary: function () {
            return this.salary;
        },
        toString: function () {
            return this.name + ''s salary is ' + this.getSalary() + '.';
        }
};

//无参无逻辑的子类构造函数
function Manager() {}

Manager.prototype = new Employee();
Manager.prototype.constructor = Manager;
//把构造逻辑搬到init方法中来
Manager.prototype.init = function (name, salary, percentage) {
    //借用父类的init方法，实现属性继承（name, salary）
    Employee.prototype.init.apply(this, [name, salary]);
    this.percentage = percentage;
};
Manager.prototype.getSalary = function () {
    return this.salary + this.salary * this.percentage;
};
```

用init方法来完成构造功能，就可以保证在设置子类原型时（Manager.prototype = new Employee()），父类的实例化操作一定不会出错，唯一不好的是在调用类的构造函数来初始化实例的时候，必须在调用构造函数后手动调用init方法来完成实际的构造逻辑：

```
var e = new Employee();
e.init('jason', 5000);
var m = new Manager();
m.init('tom', 8000, 0.15);

console.log(e.toString()); //jason's salary is 5000.
console.log(m.toString()); //tom's salary is 9200.

console.log(m instanceof Manager); //true
console.log(m instanceof Employee); //true
console.log(e instanceof Employee); //true
console.log(e instanceof Manager); //false
```

要是能把这个init的逻辑放在构造函数内部就好了，可是这样的话就会违背前面说的构造函数无参无逻辑的原则。换一种方式来考虑，这个原则的目的是为了保证在实例化父类作为子类原型的时候，调用父类的构造函数不会出错，那么就可以稍微打破一下这个原则，在类的构造函数里添加少量的并且一定不会有问题的逻辑来解决：

```
//添加一个全局标识initializing，表示是否正在进行子类的构建和类的继承
var initializing = false;
//可自动调用init方法的父类构造函数
function Employee() {
    if (!initializing) {
        this.init.apply(this, arguments);
    }
}

Employee.prototype = {
    constructor: Employee,
    //把构造逻辑搬到init方法中来
    init: function (name, salary) {
            this.name = name;
            this.salary = salary;
        },
        getName: function () {
            return this.name;
        },
        getSalary: function () {
            return this.salary;
        },
        toString: function () {
            return this.name + ''s salary is ' + this.getSalary() + '.';
        }
};

//可自动调用init方法的子类构造函数
function Manager() {
    if (!initializing) {
        this.init.apply(this, arguments);
    }
}

//表示开始子类的构建和类的继承
initializing = true;
//此时调用new Emplyee()，并不会调用Employee.prototype.init方法
Manager.prototype = new Employee();
Manager.prototype.constructor = Manager;
//表示结束子类的构建和类的继承，之后调用new Employee或new Manager都会自动调用init实例方法
initializing = false;

//把构造逻辑搬到init方法中来
Manager.prototype.init = function (name, salary, percentage) {
    //借用父类的init方法，实现属性继承（name, salary）
    Employee.prototype.init.apply(this, [name, salary]);
    this.percentage = percentage;
};
Manager.prototype.getSalary = function () {
    return this.salary + this.salary * this.percentage;
};
```

调用结果仍然和前面的例子一样。但是这个实现还有一个小问题，它引入了一个全局变量initializing，要是能把引入这个全局变量就好了，这个其实很好解决，只要我们把关于类的构建跟继承，封装成一个模块，然后把这个变量放在模块的内部，就没有问题了。

3）在构造子类的时候，是把子类的原型设置成了父类的一个实例，这个是不符合语义的，继承应该发生在类与类之间，而不是类与实例之间。之所以要用父类的一个实例来作为子类的原型：

```
SubClass.prototype = new SuperClass();
```

完全是因为父类的这个实例，指向父类的原型，而子类的实例又会指向子类的原型，所以最终子类的实例就能通过原型链访问到父类原型上的方法。这个做法虽然能实现实例方法的继承，但是它不符合语义，而且它还有一个很大的问题就是会增加原型链的长度，导致子类在调用父类方法时，必须通过原型链的查找到父类的方法才行。要是继承层次较深，会对js的执行性能有些影响。

**解决方式：**在解决这个问题之前，先想想继承能帮我们解决什么问题：从父类复用已有的实例属性和实例方法。在javascript面向对象编程中，一直有一个原则就是，实例属性都写在构造函数或者实例方法里面，实例方法写在原型上面，也就是说类的原型，按照这个原则来说，就是用来写实例方法的，而且是只用来写实例方法，那么我们完全可以在构建子类时，通过复制的方式将父类原型的所有方法全部添加到子类的原型上，不一定要把父类的一个实例设置成子类的原型，这样就能将原型链的长度大大地缩短，借助一个简短的copy函数，我们就能轻松对前面的代码进行改造：

```
//用来复制父类原型，由于父类原型上约定只写实例方法，所以复制的时候不必担心引用的问题
var copy = function (source) {
    var target = {};
    for (var i in source) {
        if (source.hasOwnProperty(i)) {
            target[i] = source[i];
        }
    }
    return target;
}

function Employee() {
    this.init.apply(this, arguments);
}

Employee.prototype = {
    constructor: Employee,
    init: function (name, salary) {
            this.name = name;
            this.salary = salary;
        },
        getName: function () {
            return this.name;
        },
        getSalary: function () {
            return this.salary;
        },
        toString: function () {
            return this.name + ''s salary is ' + this.getSalary() + '.';
        }
};

function Manager() {
    this.init.apply(this, arguments);
}
//将父类的原型方法复制到子类的原型上
Manager.prototype = copy(Employee.prototype);
//子类还是需要修改constructor指向，因为从父类原型复制出来的对象的constructor还是指向父类的构造函数
Manager.prototype.constructor = Manager;

Manager.prototype.init = function (name, salary, percentage) {
    Employee.prototype.init.apply(this, [name, salary]);
    this.percentage = percentage;
};
Manager.prototype.getSalary = function () {
    return this.salary + this.salary * this.percentage;
};

var e = new Employee('jason', 5000);
var m = new Manager('tom', 8000, 0.15);

console.log(e.toString()); //jason's salary is 5000.
console.log(m.toString()); //tom's salary is 9200.

console.log(m instanceof Manager); //true
console.log(m instanceof Employee); //false
console.log(e instanceof Employee); //true
console.log(e instanceof Manager); //false
```

这么做了以后，当调用m.toString的时候其实调用的是Manager类自身原型上的方法，而不是Employee类的实例方法，缩短了在原型链上查找方法的距离。这个做法在性能上有很大的优点，但不好的是通过原型链维持的继承关系其实已经断了，子类的原型和子类的实例都无法再通过js原生的属性访问到父类的原型，所以这个调用console.log(m instanceof Employee)输出的是false。不过跟性能比起来，这个都可以不算问题：一是instanceOf的运算，几乎在javascript的开发里面用不到，至少我是没碰到过；二是通过复制方式完全能够把父类的实例方法继承下来，这就已经达到了继承的最大目的。

这个方法还有一个额外的好处是，解决了第2个问题最后提到的引入initializing全局变量的问题，如果是复制的话，就不需要在构建继承关系时，去调用父类的构造函数，那么也就没有必要在构造函数内先判断initializing才能去调用init方法，上面的代码中就已经去掉了initializing这个变量的处理。

4）在子类的构造函数和实例方法内如果想要调用父类的构造函数或者方法，显得比较繁琐：

```
function SuperClass() {}

SuperClass.prototype = {
    constructor: SuperClass,
    method1: function () {}
}

function SubClass() {
    //调用父类构造函数
    SuperClass.apply(this);
}

SubClass.prototype = new SuperClass();
SubClass.prototype.constructor = SubClass;
SubClass.prototype.method1 = function () {
    //调用父类的实例方法
    SuperClass.prototype.method1.apply(this, arguments);
}
SubClass.prototype.method2 = function () {}
SubClass.prototype.method3 = function () {}
```

每次都得靠apply借用方法来处理。要是能改成如下的调用就好用多了：

```
function SubClass() {
//调用父类构造函数
        this.base();
}

SubClass.prototype = new SuperClass();
SubClass.prototype.constructor = SubClass;
SubClass.prototype.method1 = function() {
//调用父类的实例方法
        this.base();
}
```

**解决方式：**如果要在每个实例方法里，都能通过this.base()调用父类原型上相应的方法，那么this.base就一定不是一个固定的方法，需要在每个实例方法执行期间动态地将this.base指定为父类原型的同名方法，能够做到这个实现的方式，就只有通过方法代理了，前面的Employee和Manager的例子可以改造如下：

```
//用来复制父类原型，由于父类原型上约定只写实例方法，所以复制的时候不必担心引用的问题
var copy = function (source) {
    var target = {};
    for (var i in source) {
        if (source.hasOwnProperty(i)) {
            target[i] = source[i];
        }
    }
    return target;
};

function Employee() {
    this.init.apply(this, arguments);
}

Employee.prototype = {
    constructor: Employee,
    init: function (name, salary) {
            this.name = name;
            this.salary = salary;
        },
        getName: function () {
            return this.name;
        },
        getSalary: function () {
            return this.salary;
        },
        toString: function () {
            return this.name + ''s salary is ' + this.getSalary() + '.';
        }
};

function Manager() {
    //必须在每个实例中添加baseProto属性，以便实例内部可以通过这个属性访问到父类的原型
    //因为copy函数导致原型链断裂，无法通过原型链访问到父类的原型
    this.baseProto = Employee.prototype;
    this.init.apply(this, arguments);
}

Manager.prototype = copy(Employee.prototype);
//子类还是需要修改constructor指向，因为从父类原型复制出来的对象的constructor还是指向父类的构造函数
Manager.prototype.constructor = Manager;

Manager.prototype.init = (function (name, func) {
    return function () {
        //记录实例原有的this.base的值
        var old = this.base;
        //将实例的this.base指向父类的原型的同名方法
        this.base = this.baseProto[name];
        //调用子类自身定义的init方法，也就是func参数传递进来的函数
        var ret = func.apply(this, arguments);
        //还原实例原有的this.base的值
        this.base = old;
        return ret;
    }
})('init', function (name, salary, percentage) {
    //通过this.base调用父类的init方法
    //这个函数真实的调用位置是var ret = func.apply(this, arguments);
    //当调用Manager实例的init方法时，其实不是调用的这个函数
    //而是调用上面那个匿名函数里面return的匿名函数
    //在return的匿名函数里，先把this.base指向为了父类原型的同名函数，然后在调用func
    //func内部再通过调用this.base时，就能调用父类的原型方法。
    this.base(name, salary);
    this.percentage = percentage;
});

Manager.prototype.getSalary = function () {
    return this.salary + this.salary * this.percentage;
};

var e = new Employee('jason', 5000);
var m = new Manager('tom', 8000, 0.15);

console.log(e.toString()); //jason's salary is 5000.
console.log(m.toString()); //tom's salary is 9200.

console.log(m instanceof Manager); //true
console.log(m instanceof Employee); //false
console.log(e instanceof Employee); //true
console.log(e instanceof Manager); //false
```

通过代理的方式，就解决了在在实例方法内部通过this.base调用父类原型同名方法的问题。可是在实际情况中，每个实例方法都有可能需要调用父类的实例，那么每个实例方法都要添加同样的代码，显然这会增加很多麻烦，好在这部分的逻辑也是同样的，我们可以把它抽象一下，最后都放到模块化的内部去，这样就能简化代理的工作。

5）未考虑静态属性和静态方法。尽管静态成员是不需要继承的，但在有些场景下，我们还是需要静态成员，所以得考虑静态成员应该添加在哪里。

**解决方式：**由于js原生并不支持静态成员，所以只能借助一些公共的位置来处理。最佳的位置是添加到构造函数上：

```
var copy = function (source) {
    var target = {};
    for (var i in source) {
        if (source.hasOwnProperty(i)) {
            target[i] = source[i];
        }
    }
    return target;
};

function Employee() {
    this.init.apply(this, arguments);
}

//添加一个静态属性
Employee.idCounter = 1;
//添加一个静态方法
Employee.getId = function () {
    return Employee.idCounter++;
};

Employee.prototype = {
    constructor: Employee,
    init: function (name, salary) {
            this.name = name;
            this.salary = salary;
            //调用静态方法
            this.id = Employee.getId();
        },
        getName: function () {
            return this.name;
        },
        getSalary: function () {
            return this.salary;
        },
        toString: function () {
            return this.name + ''s salary is ' + this.getSalary() + '.';
        }
};

function Manager() {
    this.baseProto = Employee.prototype;
    this.init.apply(this, arguments);
}

Manager.prototype = copy(Employee.prototype);
Manager.prototype.constructor = Manager;

Manager.prototype.init = (function (name, func) {
    return function () {
        var old = this.base;
        this.base = this.baseProto[name];
        var ret = func.apply(this, arguments);
        this.base = old;
        return ret;
    }
})('init', function (name, salary, percentage) {
    this.base(name, salary);
    this.percentage = percentage;
});

Manager.prototype.getSalary = function () {
    return this.salary + this.salary * this.percentage;
};

var e = new Employee('jason', 5000);
var m = new Manager('tom', 8000, 0.15);

console.log(e.toString()); //jason's salary is 5000.
console.log(m.toString()); //tom's salary is 9200.

console.log(m instanceof Manager); //true
console.log(m instanceof Employee); //false
console.log(e instanceof Employee); //true
console.log(e instanceof Manager); //false
console.log(m.id); //2
console.log(e.id); //1
```

最后的两行输出了正确的实例id，而这个id是通过Employee类的静态方法生成的。在java的面向对象编程中，子类跟父类都可以定义静态成员，在调用的时候还存在覆盖的问题，在js里面，因为受语言的限制，自定义的静态成员不可能实现全面的面向对象功能，就像上面这种，能够给类提供一些公共的属性和公共方法，就已经足够了。

### 2. 期望的调用方式

从第1部分的分析可以看出，在js里面，类的构建与继承，有很多通用的逻辑，完全可以把这些逻辑封装成一个单独的模块，形成一个通用的类库，以便在工作中有需要的时候，都可以直接拿来使用。这个类库要求能完成我们需要的功能（类的构建与继承和静态成员的添加），同时在使用时要足够简洁方便。在利用bootstrap的modal组件自定义alert，confirm和modal对话框这篇文章里，我曾说过一些从组件期望的调用方式，去反推组件实现的一些观点，当你明确你需要什么东西时，你才知道这个东西你该怎么去创造。本文要编写的这个继承组件也会采取这个方法来实现，我先用前面Employee和Manager的例子来模拟调用这个继承库的场景，通过预设的一些组件名称或者接口名称以及调用方式，来尝试走通真实使用这个继承库的流程，有了这个东西，下一步我只需要根据这个要求去实现即可，模拟如下：

```
//通过调用Class函数构造一个类
var Employee = Class({
    //通过instanceMembers指定这个类的实例成员
    instanceMembers: {
        init: function (name, salary) {
                this.name = name;
                this.salary = salary;
                //调用静态方法
                this.id = Employee.getId();
            },
            getName: function () {
                return this.name;
            },
            getSalary: function () {
                return this.salary;
            },
            toString: function () {
                return this.name + ''s salary is ' + this.getSalary() + '.';
            }
    },
    //通过staticMembers指定这个类的静态成员
    //静态方法内部可通过this访问其它静态成员
    //在外部可通过Employee.getId这种方式访问到静态成员
    staticMembers: {
        idCounter: 1,
        getId: function () {
            return this.idCounter++;
        }
    }
});

var Manager = Class({
    instanceMembers: {
        init: function (name, salary, percentage) {
                this.base(name, salary);
                this.percentage = percentage;
                Manager.count++;
            },
            getSalary: function () {
                return this.salary + this.salary * this.percentage;
            }
    },
    //通过extend指定要继承的类
    extend: Employee
});
```

从模拟的结果来看，我想要的继承库对外提供的名称只有Class, instanceMembers, staticMembers和extend而已，调用方式也很简单，只要传递参数给Class函数即可。接下来就按照这个目标，看看如何一步步根据第一部分罗列的那些问题和解决方式，把这个库给写出来。

### 3. 继承库的详细实现

根据API名称和接口以及前面第1部分提出的问题，这个继承库要完成的功能有：

1）类的构建（关键：init方法）和静态成员处理；

2）继承关系的构建（关键：父类原型的复制）；

3）父类方法的简化调用（关键：父类原型上同名方法的代理）。

所以这个库的实现，可以按照这三点分成三版来开发。

1）第一版

在第一版里面，仅需要实现类的构架和静态成员添加的功能即可，细节如下：

```
var Class = (function () {
    var hasOwn = Object.prototype.hasOwnProperty;

    //用来判断是否为Object的实例
    function isObject(o) {
        return typeof (o) === 'object';
    }

    //用来判断是否为Function的实例
    function isFunction(f) {
        return typeof (f) === 'function';
    }

    function ClassBuilder(options) {
        if (!isObject(options)) {
            throw new Error('Class options must be an valid object instance!');
        }

        var instanceMembers = isObject(options) & options.instanceMembers || {},
            staticMembers = isObject(options) && options.staticMembers || {},
            extend = isObject(options) && isFunction(options.extend) && options.extend,
            prop;

        //表示要构建的类的构造函数
        function TargetClass() {
            if (isFunction(this.init)) {
                this.init.apply(this, arguments);
            }
        }

        //添加静态成员，这段代码需在原型设置的前面执行，避免staticMembers中包含prototype属性，覆盖类的原型
        for (prop in staticMembers) {
            if (hasOwn.call(staticMembers, prop)) {
                TargetClass[prop] = staticMembers[prop];
            }
        }

        TargetClass.prototype = instanceMembers;
        TargetClass.prototype.constructor = TargetClass;

        return TargetClass;
    }

    return ClassBuilder
})();
```

这一版核心代码在于类的构建和静态成员添加的部分，其它代码仅仅提供一些提前可以想到的赋值函数和变量（isObject, isFunction)，并做一些参数合法性校验的处理。添加静态成员的代码一定要在设置原型的代码之前，否则就有原型被覆盖的风险。有了这个版本，就可以直接构建带静态成员的Employee类了：

```
var Employee = Class({
    instanceMembers: {
        init: function (name, salary) {
                this.name = name;
                this.salary = salary;
                //调用静态方法
                this.id = Employee.getId();
            },
            getName: function () {
                return this.name;
            },
            getSalary: function () {
                return this.salary;
            },
            toString: function () {
                return this.name + ''s salary is ' + this.getSalary() + '.';
            }
    },
    staticMembers: {
        idCounter: 1,
        getId: function () {
            return this.idCounter++;
        }
    }
});

var e = new Employee('jason', 5000);
console.log(e.toString()); //jason's salary is 5000.
console.log(e.id); //1
console.log(e.constructor === Employee); //true
```

在getId方法中之所以直接使用this就能访问到构造函数Employee，是因为getId这个方法是添加到构造函数上的，所以当调用Employee.getId()时，getId方法里面的this指向的就是Employee这个函数对象。

第二版在第一版的基础上，实现继承关系的构建部分：

```
var Class = (function () {
    var hasOwn = Object.prototype.hasOwnProperty;

    //用来判断是否为Object的实例
    function isObject(o) {
        return typeof (o) === 'object';
    }

    //用来判断是否为Function的实例
    function isFunction(f) {
        return typeof (f) === 'function';
    }

    //简单复制
    function copy(source) {
        var target = {};
        for (var i in source) {
            if (hasOwn.call(source, i)) {
                target[i] = source[i];
            }
        }
        return target;
    }

    function ClassBuilder(options) {
        if (!isObject(options)) {
            throw new Error('Class options must be an valid object instance!');
        }

        var instanceMembers = isObject(options) & options.instanceMembers || {},
            staticMembers = isObject(options) && options.staticMembers || {},
            extend = isObject(options) && isFunction(options.extend) && options.extend,
            prop;

        //表示要构建的类的构造函数
        function TargetClass() {
            if (extend) {
                //如果有要继承的父类
                //就在每个实例中添加baseProto属性，以便实例内部可以通过这个属性访问到父类的原型
                //因为copy函数导致原型链断裂，无法通过原型链访问到父类的原型
                this.baseProto = extend.prototype;
            }
            if (isFunction(this.init)) {
                this.init.apply(this, arguments);
            }
        }

        //添加静态成员，这段代码需在原型设置的前面执行，避免staticMembers中包含prototype属性，覆盖类的原型
        for (prop in staticMembers) {
            if (hasOwn.call(staticMembers, prop)) {
                TargetClass[prop] = staticMembers[prop];
            }
        }

        //如果有要继承的父类，先把父类的实例方法都复制过来
        extend & (TargetClass.prototype = copy(extend.prototype));

        //添加实例方法
        for (prop in instanceMembers) {
            if (hasOwn.call(instanceMembers, prop)) {
                TargetClass.prototype[prop] = instanceMembers[prop];
            }
        }

        TargetClass.prototype.constructor = TargetClass;

        return TargetClass;
    }

    return ClassBuilder
})();
```

这一版关键的部分在于：

```
if(extend){
    //如果有要继承的父类
    //就在每个实例中添加baseProto属性，以便实例内部可以通过这个属性访问到父类的原型
    //因为copy函数导致原型链断裂，无法通过原型链访问到父类的原型
    this.baseProto = extend.prototype;
}
```

```
//如果有要继承的父类，先把父类的实例方法都复制过来
extend && (TargetClass.prototype = copy(extend.prototype));
```

this.baseProto主要目的就是为了让子类的实例能够有一个属性可以访问到父类的原型，因为后面的继承方式是复制方式，会导致原型链断裂。有了这一版之后，就可以加入Manager类来演示效果了：

```
var Employee = Class({
    instanceMembers: {
        init: function (name, salary) {
                this.name = name;
                this.salary = salary;
                //调用静态方法
                this.id = Employee.getId();
            },
            getName: function () {
                return this.name;
            },
            getSalary: function () {
                return this.salary;
            },
            toString: function () {
                return this.name + ''s salary is ' + this.getSalary() + '.';
            }
    },
    staticMembers: {
        idCounter: 1,
        getId: function () {
            return this.idCounter++;
        }
    }
});

var Manager = Class({
    instanceMembers: {
        init: function (name, salary, percentage) {
                //借用父类的init方法，实现属性继承（name, salary）
                Employee.prototype.init.apply(this, [name, salary]);
                this.percentage = percentage;
            },
            getSalary: function () {
                return this.salary + this.salary * this.percentage;
            }
    },
    extend: Employee
});

var e = new Employee('jason', 5000);
var m = new Manager('tom', 8000, 0.15);

console.log(e.toString()); //jason's salary is 5000.
console.log(m.toString()); //tom's salary is 9200.
console.log(e.constructor === Employee); //true
console.log(m.constructor === Manager); //true
console.log(e.id); //1
console.log(m.id); //2
```

不过在Manager内部，调用父类的方法时还是apply借用的方式，所以在最后一版里面，需要把它变成我们期望的this.base的方式，反正原理前面也已经了解了，无非是在方法同名的时候，对实例方法加一个代理而已，实现如下：

```
var Class = (function () {
    var hasOwn = Object.prototype.hasOwnProperty;

    //用来判断是否为Object的实例
    function isObject(o) {
        return typeof (o) === 'object';
    }

    //用来判断是否为Function的实例
    function isFunction(f) {
        return typeof (f) === 'function';
    }

    //简单复制
    function copy(source) {
        var target = {};
        for (var i in source) {
            if (hasOwn.call(source, i)) {
                target[i] = source[i];
            }
        }
        return target;
    }

    function ClassBuilder(options) {
        if (!isObject(options)) {
            throw new Error('Class options must be an valid object instance!');
        }

        var instanceMembers = isObject(options) & options.instanceMembers || {},
            staticMembers = isObject(options) && options.staticMembers || {},
            extend = isObject(options) && isFunction(options.extend) && options.extend,
            prop;

        //表示要构建的类的构造函数
        function TargetClass() {
            if (extend) {
                //如果有要继承的父类
                //就在每个实例中添加baseProto属性，以便实例内部可以通过这个属性访问到父类的原型
                //因为copy函数导致原型链断裂，无法通过原型链访问到父类的原型
                this.baseProto = extend.prototype;
            }
            if (isFunction(this.init)) {
                this.init.apply(this, arguments);
            }
        }

        //添加静态成员，这段代码需在原型设置的前面执行，避免staticMembers中包含prototype属性，覆盖类的原型
        for (prop in staticMembers) {
            if (hasOwn.call(staticMembers, prop)) {
                TargetClass[prop] = staticMembers[prop];
            }
        }

        //如果有要继承的父类，先把父类的实例方法都复制过来
        extend & (TargetClass.prototype = copy(extend.prototype));

        //添加实例方法
        for (prop in instanceMembers) {

            if (hasOwn.call(instanceMembers, prop)) {

                //如果有要继承的父类，且在父类的原型上存在当前实例方法同名的方法
                if (extend & isFunction(instanceMembers[prop]) && isFunction(extend.prototype[prop])) {
                    TargetClass.prototype[prop] = (function (name, func) {
                        return function () {
                            //记录实例原有的this.base的值
                            var old = this.base;
                            //将实例的this.base指向父类的原型的同名方法
                            this.base = extend.prototype[name];
                            //调用子类自身定义的实例方法，也就是func参数传递进来的函数
                            var ret = func.apply(this, arguments);
                            //还原实例原有的this.base的值
                            this.base = old;
                            return ret;
                        }
                    })(prop, instanceMembers[prop]);
                } else {
                    TargetClass.prototype[prop] = instanceMembers[prop];
                }
            }
        }

        TargetClass.prototype.constructor = TargetClass;

        return TargetClass;
    }

    return ClassBuilder
})();
```

核心部分是：

```
if (hasOwn.call(instanceMembers, prop)) {

    //如果有要继承的父类，且在父类的原型上存在当前实例方法同名的方法
    if (extend & isFunction(instanceMembers[prop]) && isFunction(extend.prototype[prop])) {
        TargetClass.prototype[prop] = (function (name, func) {
            return function () {
                //记录实例原有的this.base的值
                var old = this.base;
                //将实例的this.base指向父类的原型的同名方法
                this.base = extend.prototype[name];
                //调用子类自身定义的实例方法，也就是func参数传递进来的函数
                var ret = func.apply(this, arguments);
                //还原实例原有的this.base的值
                this.base = old;
                return ret;
            }
        })(prop, instanceMembers[prop]);
    } else {
        TargetClass.prototype[prop] = instanceMembers[prop];
    }
}
```

只有当需要继承父类，且父类原型中有方法与当前的实例方法同名时，才会去对当前的实例方法添加代理。更详细的原理可以回到文章第1部分回顾相关内容。至此，我们在Manager类内部调用父类的方法时，就很简单了，只要通过this.base即可：

```
var Employee = Class({
    instanceMembers: {
        init: function (name, salary) {
                this.name = name;
                this.salary = salary;
                //调用静态方法
                this.id = Employee.getId();
            },
            getName: function () {
                return this.name;
            },
            getSalary: function () {
                return this.salary;
            },
            toString: function () {
                return this.name + ''s salary is ' + this.getSalary() + '.';
            }
    },
    staticMembers: {
        idCounter: 1,
        getId: function () {
            return this.idCounter++;
        }
    }
});

var Manager = Class({
    instanceMembers: {
        init: function (name, salary, percentage) {
                //通过this.base调用父类的构造方法
                this.base(name, salary);
                this.percentage = percentage;
            },
            getSalary: function () {
                return this.base() + this.salary * this.percentage;
            }
    },
    extend: Employee
});

var e = new Employee('jason', 5000);
var m = new Manager('tom', 8000, 0.15);

console.log(e.toString()); //jason's salary is 5000.
console.log(m.toString()); //tom's salary is 9200.
console.log(e.constructor === Employee); //true
console.log(m.constructor === Manager); //true
console.log(e.id); //1
console.log(m.id); //2
```

注意这两处调用：

```
var Manager = Class({
    instanceMembers: {
        init: function (name, salary, percentage) {
                //通过this.base调用父类的构造方法
                this.base(name, salary);//要注意的第一处
                this.percentage = percentage;
            },
            getSalary: function () {
                return this.base() + this.salary * this.percentage;//要注意的第二处this.base()
            }
    },
    extend: Employee
});
```

以上就是本文要实现的继承库的全部细节，其实它所做的事就是把本文第1部分提到的那些问题的解决方式和第二部分模拟的调用场景结合起来，封装到一个模块内部而已，各个细节的原理只要理解了第1部分总结的那些解决方式就很掌握了。在最后一版的演示中，也能看到，本文实现的这个继承库，已经完全满足了模拟场景中的需求，今后有任何需要用到继承的场景，完全可以拿最后一版的实现去开发。

### 4. 总结

本文在三生石上关于javascript继承系列博客的思路指引下，实现了一个易用的继承库，使用它可以更像java语言构建面向对象的类和类之间的继承关系，我可以预见在将来的工作，这个库对我的代码质量和功能实现会起到很重要的作用，因为在开发中，继承的编码思想还是应用的非常多，尤其是当我们做项目做得多的时候，一方面肯定想把一些公共的东西写成可重用的组件，另一方面又必须得满足各个项目的个性要求，所以在写组件的时候不能写的太死，多写接口，等到具体项目的时候再通过继承等方式来扩展该项目独有的功能，这样写出的组件才会更灵活稳定。总之有了这个继承库，感觉以后写的代码都会开心好多~所以希望本文的内容也能对你有同样的一些帮助。如果确实有帮助，求点推荐：）

谢谢阅读！

文章转载：[http://www.cnblogs.com](http://www.cnblogs.com/lyzg/p/5313752.html)
