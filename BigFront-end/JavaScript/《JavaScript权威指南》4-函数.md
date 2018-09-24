# 8 函数

函数是一段JavaScript代码，只定一次，但可能被运行任意次。除了**实参**外，每一次函数调用还有另一个值——**调用的上下文——即this。**


---
## 8.1 定义函数与函数调用

### 定义函数

- 函数的定义方式有三种
- 只运行一次时，可以定义匿名函数
- 函数可以嵌套声明，但不能出现在循环、条件判断、`try/catch`中

```javascript
//定义函数
var funcA = function (a, b, c) {
    return a + b + c;
};

//定义函数
function funcB(a, b) {
    return a / b;
}

//创建函数对象
var funcC = new Function("var a = 10; var b = 20; return a+b;");
console.log(funcC());

//嵌套函数
function outerFunc(a) {
    function innerFunc(b) {
        b.call();
    }

    innerFunc(a);
}

//匿名函数
outerFunc(function () {
    console.log("I am running");//->I am running
});

//定义函数立即调用
(function (a, b) {
    console.log(a + b);
})(1, 2);//->3
```


### 函数的调用方式

函数的调用方式有四种：

- 作为函数调用：ES3和非严格模式下的ES5中，函数调用的上下文(this)是全局对象，**ES5的严格模型下是undefined**。
- 作为方法调用：方法调用即调用对象上的方法，方法调用的上下文(this)是方法所属的对象，但是方法中嵌套函数的调用的上下文不是外侧函数，而是全局对象(或者undefined)。
- 作为构造函数：构造函数调用即在构造函数或者方法调用前加上new关键字。构造函数创建的对象继承构造函数的prototype。
- 通过它们的`call和apply方法`间接调用：js中的函数也是对象，和其他js对象没什么两样。函数也可以包含方法，比如：`call()和apply()`。

```javascript
//Function
function Fa() {
    console.log(this);//->Fa {}
}
new Fa();
```

---
## 8.2 函数的实参与形参

- 形参是可选的，多出来的形参被定义为undefined，多余的实参会被忽略
- 标识符`arguments`指向实参对象的引用
- 当函数的形参过多时，可以将多个参数包装成一个对象
- `arguments`中定义了`callee`和`caller`属性，在ES5中的严格模式下，访问这两个属性都会产生类型错误，非严格模式下，callee代表正在执行的函数，caller是非标准的。代表调用正在执行函数的函数
- 在严格模式下：arguments是一个保留字，无法使用arguments作为形参名，或者局部遍历名，也不能给arguments赋值
- arguments是一个类数组对象，其有length属性

```javascript
//可选形参，optional表明b是可选的
function getProperNames(a, /*optional*/b) {
    if (b == null) {
        b = [];
    }
    for (var property in  a) {
        b.push(property);
    }
    return b;
}

//检查参数的个数
function f(x, y, z) {
    if (arguments.length !== 3) {
        throw new Error("function f called with arguments.length must is 3");
    }
    return x + y + z;
}

//可变长参数
function max(/* ... */) {
    var max = Number.NEGATIVE_INFINITY;
    for (var i = 0, length = arguments.length; i < length; i++) {
        if (max < arguments[i]) {
            max = arguments[i];
        }
    }
    return max;
}

//arguments中定义了callee和caller属性，在ES5中的严格模式下，访问这两个属性都会产生类型错误
//非严格模式下，callee代表正在执行的函数，caller是非标准的。代表调用正在执行函数的函数
//在严格模式下：arguments是一个保留字，无法使用arguments作为形参名，或者局部遍历名，也不能给arguments赋值
function testCallee() {
    console.log(arguments.callee);
    console.log(arguments.caller);
}


//参数过多时，对象属性作为实参
function arrayCopy(/*array*/from, /*index*/from_start, /*array*/to, /*index*/to_start, /*integer*/length) {
    //logic fake
    for (var a in from) {
        to.push(a);
    }
    return to;
}

function easyCopy(args) {
    var result = arrayCopy(args.from, args.from_start || 0, args.to, args.to_start || 0, args.length);
    return result;
}
```

---
## 8.3 作为命名空间的函数

作为命名空间的函数的应用：在函数中声明的变量在整个函数体内都是可见的(包括在嵌套的函数中)，在函数的外部是不可见的。一般情况下，无法在js中声明只在一个代码块内可见的变量，基于此原因，我们常常简单的定义一个函数作临时的命名空间，在这个命名空间内定义的变量都不会污染的全局命名空间。

实例：
```javascript
// 特定场景下返回带补丁的extend版本：
// 修复多数IE中的bug：如果o的属性拥有一个不可枚举的同名属性，则for/in循环不会枚举对象o的可枚举属性， 也就是说，如果对象o有一个不可枚举的toString属性，则对象o的toString方法不会被正确的处理。

var extend = (function () { //此函数的返回值赋值给extend
    //修复前，检查是否存在bug
    for (var p in {toString: null}) {
        //代码执行到这里，说明for/in循环正确的工作并返回，则返回一个简单版本的函数
        return function extend(o) {
            for (var i = 1; i < arguments.length; i++) {
                var source = arguments[i];
                for (var prop in source) {
                    o[prop] = source[prop];
                }
            }
            return o;
        }
    }

    var protoprops = ['toString', 'valueOf', 'constructor', 'hasOwnProperty', 'isPrototypeOf', 'propertyIsEnumerable', 'toLocaleString'];

    /*
     代码执行到这里，说明for/in不会枚举对象的toString属性
     因此返回另一个版本的extends函数，这个函数显式测试Object.prototype中不可枚举的属性
     */
    return function patched_extend(o) {
        for (var i = 1; i < arguments.length; i++) {
            var source = arguments[i];
            //赋值所有可枚举的属性
            for (var prop in source) {
                o[prop] = source[prop];
            }
            //检查特殊名称的属性
            for (var j = 0; j < protoprops.length; j++) {
                prop = protoprops[j];
                if (source.hasOwnProperty(prop)) {
                    o[prop] = source[prop];
                }
            }
        }
        return o;
    };
}());

//按照这种写法，patched_extend方法就不会污染到全局环境
var result = extend({}, {x: 1}, {y: 2, toString: 32});

console.log(result);//->{ x: 1, y: 2, toString: 32 }
console.log(result.toString);//->32
console.log(Object.prototype.toString.call(result));//->[object Object]
```

---
## 8.4 闭包

函数的执行依赖变量的作用域，这个作用域是在函数定义时确定的，而不是函数调用时确定的，为了实现这种词法作用域，js函数对象的内部状态不仅包括函数的代码逻辑，还必需引用函数当前的作用域链。函数对象可以通过作用域链关联起来，函数体内部的变量都可以保存在函数作用域内，这种特性在计算机科学中称为**闭包**。

> 维基百科：在计算机科学中，闭包（英语：Closure），又称词法闭包（Lexical Closure）或函数闭包（function closures），**是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。**所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的**实体**。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

从技术较大角度来讲，js中所有的函数都是闭包，它们都是对象，它们都关联到作用域链。

**闭包的实现**：
>如果一个函数的局部变量是定义在cpu的栈中，那么当函数返回时它们就不存在了。但是js的采用作用域链，每一次函数调用都为之创建一个新的对象来保存局部变量，把这个对象添加到作用域链中。当函数返回就从作用域链中删除这个绑定变量的对象。 如果不存在嵌套函数，也没有其他引用指向这个绑定的对象，它就会被当作垃圾回收掉， 如果定义了嵌套函数，每个嵌套的函数都有各自对应的作用域链，并且这个作用域链指向
 一个变量绑定对象，但如果这些嵌套函数在外部函数中保存下来，那么它们也会和所指向的变量绑定对象一样当作垃圾被回收，当如果这个函数定义了嵌套函数，并将它作为返回值返回，或者存储在某处的属性里，这时就会有一个外部引用指向这个嵌套的函数，他就不会被当作垃圾回收，并且它所指向的变量绑定对象也不会当作垃圾回收。

```javascript
var scope = "global scope";
function checkScope() {
    var scope = "local scope";

    function f() {
        return scope;
    }

    return f();
}

var result = checkScope();
console.log(result);//->local scope

//使用闭包实例：计数器，同时有很好的隐藏了counter变量
var uniqueInteger = (function () {
    var counter = 0;
    return function () {
        return counter++;
    }
}());
console.log(uniqueInteger());//->0
console.log(uniqueInteger());//->1

//计数器2
function counter() {
    var n = 0;
    return {
        count: function () {
            return n++;
        },
        reset: function () {
            n = 0;
        }
    }
}
var c = counter();
console.log(c.count());//->0
console.log(c.count());//->1


//技术角度来讲，可以将闭包合并为属性存储器方法getter和setter
function counter2(n) {
    return {
        get count() {
            return n++;
        },
        set count(value) {
            if (value >= n) {
                n = value;
            } else {
                throw Error("count can only be set to a larger value");
            }
        }
    }
}

var c = counter2(1000);
console.log(c.count);//->1000
console.log(c.count);//->1001
```

闭包访问this和arguments：

1. 闭包在外部函数里无法访问this，除非外部函数将this转存为一个变量：`var self = this;`
2. 闭包有自己的argument参数，所以闭包用样无法直接访问外部函数的参数数组

---
## 8.5 函数属性、方法和构造函数

### 要点

函数也是对象，只是`typeof`操作返回“function”，所以函数也有属性和方法。

- `length`属性：函数的只读属性，表示函数形参的个数
- `protetype`属性：函数的原型对象
- `call`方法和`apply`方法
    - ES5中，`call`方法和`apply`方法的第一个实参都会变为this的值，哪怕传入的是null或者undefined。在ES3或者ES5非严格模式下，传入的是null或者undefined会被替换为全局对象，其他原始值都被相应的包装对象所替代。
    - 对于call方法，除了第一个参数变为this外，其他参数都是函数的参数
    - 对于apply方法，除了第一个参数变为this外，只接受一个数组(也可以是类数组对象)作为函数的参数。
- `bind`方法是在ES5中新增的方法。主要作用是将函数绑定到某个对象。bind方法返回一个新的函数。对于bind方法，除了第一个参数之外，传入bind方法的实参也会绑定至this，可用于实现科里化闭包。
- `toString`方法一般返回函数的全部代码
- Function构造函数，一般很少使用，所有函数的constructor属性是Function

```javascript
//length
function testLength(a, b, c) {

}
var length = testLength.length;
console.log(length);//->3

//apply方法和call方法
var o = [1, 2, 3];
function applyFunc(a, b) {
    console.log("this: " + this);//->o
    console.log(a);//->{x: 1}
    console.log(b);//->{y:2}]
}
applyFunc.apply(o, [{x: 1}, {y: 2}]);//o此时时指定调用函数的上下文
applyFunc.call(o, {x: 1}, {y: 2});//o此时时指定调用函数的上下文
```

### bind方法

`bind`方法是在ES5中新增的方法。主要作用是将函数绑定到某个对象。bind方法返回一个新的函数。对于bind方法，**除了第一个参数之外，传入bind方法的实参也会绑定至this**，可用于实现科里化闭包。


```javascript
//bind方法
function f1(y) {
    return this.x + y;
}
var o = {x: 1};
var g = f1.bind(o);
console.log(g(1));//->2


var sum = function (x, y) {
    return x + y;
}
var succ = sum.bind(null, 1);

console.log(succ(2));//->3
function f2(y, z) {
    return this.x + y + z;
}
var g = f2.bind({x: 1}, 2);//f2科里化，y被绑定尾2
console.log(g(3));//->6


//========================================================================
// es3版本模拟bind
//========================================================================
f (!Function.prototype.bind) {
    Function.prototype.bind = function (o/*, args*/) {
        var self = this;
        var boundArgs = arguments;
        return function () {
            var args = [], i;
            for (i = 1; i < boundArgs.length; i++) {
                args.push(boundArgs[i]);
            }
            for (i = 0; i < arguments.length; i++) {
                args.push(arguments[i]);
            }
            return self.apply(o, args);
        }
    }
}

//测试模拟的bind方法
Function.prototype.mockBind = function (o/*, args*/) {
    //this指向调用mockBind方法的函数，如下面的testMockBind
    var self = this;
    var boundArgs = arguments;
    return function () {
        var args = [], i;
        for (i = 1; i < boundArgs.length; i++) {//外函数的参数
            args.push(boundArgs[i]);
        }
        for (i = 0; i < arguments.length; i++) {//本函数的参数
            args.push(arguments[i]);
        }
        //把o绑定为this，调用原函数
        return self.apply(o, args);
    }
};

function testMockBind(a) {
    console.log(a);
    console.log(this);//->{ x: 1, y: 2, z: 2 }
    return this.x + this.y + a;
}

var newFunction1 = testMockBind.mockBind({x: 1, y: 2, z: 2}, 15);//上面a=15
var newFunction2 = testMockBind.mockBind({x: 1, y: 2, z: 2});//上面a=12
console.log(newFunction1(100));//->18，而这里的100被忽略
console.log(newFunction2(12));//->15
```

>我们注意到，模拟的 bind()方法返回的函数是一个闭包，在这个闭包的外部函数中声明了self和boundArgs变量，这两个变量在闭包里用到。尽管定义闭包的内部函数已经从外部函数中返回，而且调用这个闭包逻辑的时刻要在外部函数返回之后（在闭包中照样可以正确访问这两个变量）。ECMAScript 5定义的bind()方法也有一些特性是上述ECMAScript 3代码无法模拟的。首先，真正的bind()方法返回一个函数对象，这个函数对象的length属性是绑定函数的形参个数减去绑定实参的个数（length的值不能小于零），再者，ECMAScript 5的bind()方法可以顺带用做构造函数。如果bind返回的函数用做构造函数，将忽略传入bind的this，原始函数就会以构造函数的形式调用，其实参也已经绑定。**由bind方法所返回的函数并不包含prototype属性**（普通函数固有的prototype属性是不能删除的），并且将这些绑定的函数用做构造函数时所创建的对象从原始的未绑定的构造函数中继承prototype。同样，在使用instanceof运算符时，绑定构造函数和未绑定构造函数并无两样。


### Function构造函数

Function构造函数可以传人任意数量的字符串实参，最后一个实参所表示的文本就是函数体；它可以包含任意的JavaScript语句，每两条语句之间用分号分隔。传入构造函数的其他所有的实参字符串是指定函数的形参名字的字符串。如果定义的函数不包含任何参数，只须给构造函数简单地传入一个字符串`函数体`即可。

注意，FunctionO构造函数并不需要通过传入实参以指定函数名。就像函数直接量一样，FunctionO构造函数创建一个匿名函数。 关于Function ()构造函数有几点需要特别注意：

- `Function ()`构造函数允许JavaScript在运行时动态地创建并编译函数。
- 每次调用FunctionO构造函数都会解析函数体，并创建新的函数对象。如果是在一个循环或者多次调用的函数中执行这个构造函数，执行效率会受影响。相比之下，循环中的嵌套困数和函数定义表达式则不会每次执行时都重新编译。
- 最后一点，也是关于Function构造函数非常重要的一点，就是它所创建的函数并不是使用词法作用域，相反，函数体代码的编译总是会在顶层函数执行，正如下面代码所示：

```javascript
var scope = "global"
function constructFunction() {
   var scope = "local";
   return new Function ("return scope"); //无法捕获局部作用域
}

//这一行代码返回global,因为通过Function()构造函数
//所返回的函数使用的不是局部作用域
constructFunction()(); //-> "global"
```

我们可以将`Function()`构造函数认为是在全局作用域中执行的eval()，`eval()`可以在自己的私有作用域内定义新变量和函数，`Function()`构造函数在实际编程中很少用到

###  可调用对象

可调用对象：可以像函数一样被调用的对象。函数是可以被调用的，但是可以调用的并不一定就是函数。

### 使用函数自身的属性来缓存结果

```javascript
//计算阶乘
function factorial(n) {
    if (isFinite(n) && n > 0 && n === Math.round(n)) {//有限的正整数
        if (!(n in factorial)) {
            factorial[n] = n * factorial(n - 1);
        }
        return factorial[n];
    }
    return NaN;
}

factorial[1] = 1;//初始化

console.log(factorial(100));//->9.33262154439441e+157
console.log(factorial(2));//->2
console.log(factorial(5));//->120
```

---
## 8.6 函数式编程

JavaScript并非函数式编程语言，但是js可以像操作对象一样操作函数，因此可以在js中实现函数式编程。

### 使用函数处理数组

```javascript
var data = [1, 1, 2, 3, 4, 5, 5];
var sum = function (x, y) {
    return x + y
};
var square = function (x) {
    return Math.pow(x, 2)
};

var mean = data.reduce(sum) / data.length;//平均值
var deviations = data.map(function (p1) {
    return p1 - mean;
});//偏差

var stddev = Math.sqrt(deviations.map(square).reduce(sum) / (data.length - 1));//标准偏差

console.log(mean);//->3
console.log(deviations);//->[ -2, -2, -1, 0, 1, 2, 2 ]
console.log(stddev);//->1.7320508075688772
```


### 高阶函数

高阶函数：所谓高阶函数就是操作函数的函数

```javascript
//========================================================================
// 模拟map
//========================================================================
var map = Array.prototype.map ? function (a, f) {
    return a.map(f);
} : function (a, f) {
    var results = [];
    for (var i = 0, len = a.length; i < len; i++) {
        if (i in a) {
            results[i] = f.call(null, a[i], a);
        }
    }
    return results;
};

var result = map([1, 2, 3, 4, 5], function (x) {
    return x - 1;
});
console.log(result);//->[ 0, 1, 2, 3, 4 ]


//========================================================================
// 高阶函数：所谓高阶函数就是操作函数的函数
//========================================================================
function not(f) {
    return function () {
        var result = f.apply(this, arguments);
        return !result;
    }
}

function even(x) {
    return x % 2 === 0;
}
var odd = not(even);
console.log(odd(2));//->false

```


### 偏函数

把一次完整的函数调用拆成多次函数调用，没错传入的实参都是完整实参的一部分，每一个拆分开的函数都叫做偏函数(partial function)。当函数的参数个数太多，需要简化时，使用**偏函数**可以创建一个新的函数，这个新函数可以固定住原函数的部分参数，从而在调用时更简单。

判断类型示例：

```javascript
var isType = function(type){
  return function(obj){
    return toString.call(obj)=='[object '+type+']';
  }
};

// 定制新的函数
var isString = isType('String');
var isArray = isType('Array');
var isFunction = isType('Function');

// 测试偏函数
console.log(isString('abc')); // true
console.log(isArray([1, 2])); // true
console.log(isFunction('abc')); // false
```


《JavaScript权威指南》中的一些应用：

```javascript
function array(a, n) {
    //去掉第一个参数，因为下面partial function的第一个参数都是一个函数
    return Array.prototype.splice.call(a, n || 0);
}

//这个函数的实参传递到左边
function partialLeft(f/*, ...*/) {
    var args = arguments;
    return function () {
        var a = array(args, 1);
        a = a.concat(array(arguments));
        return f.apply(this, a);
    }
}

//这个函数的实参传递到右边
function partialRight(f/*, ...*/) {
    var args = arguments;
    return function () {
        var a = array(arguments);
        a = a.concat(array(args, 1));
        return f.apply(this, a);
    }
}

//这个函数的实参被用做模板，实参列表中的undefined值都被填充
function partial(f/*,...*/) {
    var arg = arguments;
    return function () {
        var a = array(arg, 1);
        var i = 0, j = 0;
        for (; i < a.length; i++) {
            if (a[i] === undefined) {
                a[i] = arguments[j++];
            }
        }
        //剩下的背部实参加入进去
        a = a.concat(array(arguments, j));
        return f.apply(this, a);
    }
}

var f = function (x, y, z) {
    console.log(x + " " + y + " " + z);
    return x * (y - z);
};

var left = partialLeft(f, 2)(3, 4); //2 * (3-4)
var right = partialRight(f, 2)(3, 4); //3 * (4-2)
var value = partial(f, undefined, 2)(3, 4);// 3 * (2-4)

console.log(left);//->-2
console.log(right);//->6
console.log(value);//->-6

//partialLeft、partialRight、partial被称为不完全函数，利用这种不完全函数可以编写一些有意思的代码

function sum(a, b) {
    return a + b;
}

var increment = partialLeft(sum, 1);//自增
var cubeRoot = partialRight(Math.pow, 1 / 3);//立方根
String.prototype.first = partial(String.prototype.charAt, 0);
String.prototype.last = partial(String.prototype.substr, -1, 1);

console.log(increment(1));//->2
console.log(cubeRoot(27));//->3
console.log("abc".first());//->a
console.log("abc".last());//->c

//利用不完全函数重写组织求平均数和标准差的代码
var data = [1, 1, 3, 5, 5];

var reduce = function (a, f, initial) {
    if (arguments.length > 2) {
        return a.reduce(f, initial);
    } else {
        return a.reduce(f);
    }
};//抽出array的reduce方法

var product = function (x, y) {
    return x * y;
};//积

var neg = partial(product, -1);//求负数
var square = partial(Math.pow, undefined, 2);//平方
var sqrt = partial(Math.pow, undefined, .5);//开平方根
var reciprocal = partial(Math.pow, undefined, -1);//倒数

var mean = product(reduce(data, sum), reciprocal(data.length));//平均值
console.log(mean);//->3
```

### 记忆

记忆：缓存函数的执行结果在函数式编程中称为**记忆**(memorization)

```javascript
//返回带有记忆功能的函数
function memorize(f) {
    var cache = {};//将值保存在闭包中
    return function () {
        var key = arguments.length + Array.prototype.join.call(arguments, ",");
        if (key in cache) {
            return cache[key];
        } else {
            return cache[key] = f.apply(this, arguments);
        }
    }
}
```