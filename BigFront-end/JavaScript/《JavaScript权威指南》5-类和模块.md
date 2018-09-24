
---
# 9 类和模块

js中类的一个重要特性是**动态可继承**，这适用于一种编程哲学——鸭子辩型

---
## 9.1 类和原型

js中，类的所有实例都从同一个原形对象上继承属性，因此**原型对象是类的核心**。

下面是一个例子，自定义一个`inherit`方法，用于创建一个新的对象。而传入的函数将绑定为新对象的原型。所以新对象也继承了原型上的属性和方法。

```javascript
/**
 * 传入一个原型p，返回一个新的对象，该对象的__proto__指向p
 * @param p 原型
 * @returns {*} 新的对象
 */
function inherit(p) {
    if (p == null) {
        throw TypeError();
    }
    if (Object.create) {
        return Object.create(p);
    }
    var t = typeof p;
    if (t !== "object" && t !== "function") {
        throw TypeError();
    }

    function f() {
    }

    f.prototype = p;
    return new f();
}

// 工厂方法，返回一个Range实例
function range(from, to) {
    var r = inherit(range.methods);
    r.from = from;
    r.to = to;
    return r;
}

//range的所有实例都继承这些方法，methods将在inherit方法中作为新对象的原形
range.methods = {
    includes: function (x) {
        return this.from <= x && x <= this.to;
    },
    foreach: function (f) {
        for (var x = Math.ceil(this.from); x <= this.to; x++) f(x);
    },
    toString: function () {
        return "(" + this.from + "..." + this.to + ")";
    }
};

var r = range(1, 3);
r.includes(2);
r.foreach(console.log);
console.log(r);//->{ from: 1, to: 3 }
console.log(r.constructor);//->[Function: Object] 即 Object
```

---
## 9.2 类和构造函数

构造函数是用来初始化新创建的对象的，使用new关键字调用构造函数会自动创建一个新的对象，因此构造函数只需要初始化对象的状态即可，调用构造函数的一个重要特征是，**构造函数的prototype属性会作为新创建对象的原形**。并且构造函数中的this引用将被绑定到新创建的对象。

```javascript
//定义构造函数
function Range(from, to) {
    this.from = from;
    this.to = to;
}

//定义新的原形，并且给原形添加属性和方法
Range.prototype = {
    includes: function (x) {
        return this.from <= x && x <= this.to;
    },
    foreach: function (f) {
        for (var x = Math.ceil(this.from); x <= this.to; x++) f(x);
    },
    toString: function () {
        return "(" + this.from + "..." + this.to + ")";
    }
};

//创建一个Range类
var r = new Range(1, 3);
r.includes(2);
r.foreach(console.log);
console.log(r);//->{ from: 1, to: 3 }
```

### 构造函数和类标识

**原形对象是类的唯一标识，当且仅当两个对象继承同一个原形时，它们才是属于同一个类的实例。**而初始化对象的状态的构造函数则不能作为类的标识。

- `instanceof`并不会检测对象是否由构造函数初始化而来，而会检测对象是否继承某个原形。
- 任何一个js函数都可以作为构造函数，并每一个js函数都拥有一个prototype属性(**ES5中的bind方法返回的函数除外**)，这个属性值是一个对象，这个对象包含一个唯一不可枚举的属性constructor，这个constructor是一个函数对象，指向该js函数本身。

---
## 9.3 JavaScript中Java式的类继承

类似Java中，类成员的模样应该是：

- 实例字段：它们是基于实例的属性或变量，用以保存独立对象的状态。
- 实例方法：它们是类的所有实例所共享的方法，由每个独立的实例调用。
- 类字段：这些属性或变量是属于类的，而不是属于类的某个实例的。
- 类方法：这些方法是属于类的，而不是属于类的某个实例的。

与Java不同，js中的函数都是以值的形式体现的。方法和字段之间并没有太大的区别。尽管存在诸多差异，但是依然可以在js中模拟java的这四种类成员类型。JavaScript中的类牵扯三种不同的对象，三种对象的属性的行为和下面三种类成员非常相似：

- 构造函数对象：构造函数（对象）为JavaScript的类定义了名字，任何添加到这个构造函数对象中的属性都是类字段和类方法
- 原型对象：原型对象的属性被类的所有实例所继承，如果原型对象的属性值是函数的话，这个函数就作为类的实例的方法来调用
- 实例对象：类的毎个实例都是一个独立的对象，直接给这个实例定义的属性是不会为所有实例对象所共享的。定义在实例上的非函数属性，实际上是实例的字段

在js中定义类的步骤可以分为三个步骤：

1. 先定义一个构造函数，并设置初始化新对象的实例属性
2. 给构造函数的prototype对象定义实例方法
3. 给构造函数定义类字段和类属性

模拟定义Java类与类的继承，将这三个步骤定义为defineClass方法：

```javascript
var extend = (function () {
    for (var p in {toString: null}) {
        return function extend(o) {
            for (var i = 1; i < arguments.length; i++) {
                var source = arguments[i];
                for (var prop in source) {
                    o[prop] = source[prop];
                }
            }
            return o;
        };
    }
    return function patched_extend(o) {
        for (var i = 1; i < arguments.length; i++) {
            var source = arguments[i];
            // Copy all the enumerable properties
            for (var prop in source) {
                o[prop] = source[prop];
            }

            for (var j = 0; j < protoprops.length; j++) {
                prop = protoprops[j];
                if (source.hasOwnProperty(prop)) o[prop] = source[prop];
            }
        }
        return o;
    };
    var protoprops = ["toString", "valueOf", "constructor", "hasOwnProperty", "isPrototypeOf", "propertyIsEnumerable", "toLocaleString"];
}());


//工厂方法，定义一个类
function defineClass(constructor, // 用以设置实例的属性的函数
                     methods,     //实例的方法，复制到原型中
                     statics)     // 类属性，复制到构造函数中
{
    if (methods) extend(constructor.prototype, methods);
    if (statics) extend(constructor, statics);
    // Return the class
    return constructor;
}

//创建一个SimpleRange类
var SimpleRange = defineClass(
    function (f, t) {
        this.f = f;
        this.t = t;
    },
    {
        includes: function (x) {
            return this.f <= x && x <= this.t;
        },
        toString: function () {
            return "(" + this.f + "..." + this.t + ")";
        }
    },
    {
        upto: function (t) {
            return new SimpleRange(0, t);
        }
    }
);

//创建对象并使用
var sr = new SimpleRange(1, 100);
console.log(sr.includes(3));//-> true
var sr2 = SimpleRange.upto(8);
console.log(sr2);//->{ f: 0, t: 8 }
```

---
## 9.4 类的扩充

对象从其原形继承属性，如果某个原形的属性发生变化，那么也会影响到继承这个原形的所有实例。

js中对象的原形**包括内置对象**的原形都是非常**开放**的，可以在原形上添加任意属性。但是不建议在Object.prototype上添加属性，因为在ES5之前无法创建不可枚举的属性。


---
## 9.5 类和类型

javascript中定义了少量的数据类型：`null，undefined，布尔值，数字，，字符串，函数，对象`，使用typeof可以获取值的类型，但我们往往**更希望将类作为类型来对待，这样就可以根据对象所属类型来区分它们**。

三种检测任意对象的类的技术：

1. instanceof元算符和isPrototypeOf方法：只能判断对象是否属于指定的类名，无法获取对象的类名。instanceof期望作操作数是待检测的类型，右操作数是定义类的构造函数，如果o继承自c.prototype，则`o instanceof c`返回ture
2. constructor属性：constructor是类的共有标识，但**并不是所有的的对象都具有constructor属性**。
3. 构造函数名称：使用构造函数的名称而不是构造函数本身作为类的标识符。但并不是所有的的对象都具有constructor属性。也就是不是所有的函数都有名字。

方法1和方法2在多窗口和多框架的子页面Web应用中兼容性不佳。因为每个窗口和框架子页面都具有单独的执行上下文。每个上下文都包含独有的全局变量和一组构造函数。

```javascript
//返回一个字符串，表示对象的类型
function type(o) {
    var t, c, n;  // type, class, name
    //null盘但
    if (o === null) {
        return "null";
    }
    //NaN类型
    if (o !== o) {
        return "nan";
    }
    //typeof不是object的话，那么应该是基本类型
    if ((t = typeof o) !== "object") {
        return t;
    }
    //如果对象的类信息不是Object，那么直接返回该类型
    if ((c = classOf(o)) !== "Object") {
        return c;
    }
    //如果对象具有构造函数并且构造函数具有名称，那么返回这个名称
    if (o.constructor && typeof o.constructor === "function" && (n = o.constructor.getName())) {
        return n;
    }
    //否则只能返回Object
    return "Object";
}

//返回对象的类信息
function classOf(o) {
    return Object.prototype.toString.call(o).slice(8, -1);
}

//返回函数的名称，可能为""或者null
Function.prototype.getName = function () {
    if ("name" in this){
        return this.name;
    }
    return this.name = this.toString().match(/function\s*([^(]*)\(/)[1];
};

//========================================================================
// test
//========================================================================
var number = 1;
var string = "32";
var date = new Date();
var reg = new RegExp();
var obj = {x: 2};
var array = [1, 2, 3];
var noNameFunc = function (x, y) {
    return x + y;
};
var nameFunc = function add(x, y) {
    return x + y;
};
var nonNameFuncObj = new noNameFunc(1, 3);
var nameFuncObj = new nameFunc(1, 3);

console.log(type(number));//->number
console.log(type(string));//->string
console.log(type(date));//->Date
console.log(type(reg));//->RegExp
console.log(type(obj));//->Object
console.log(type(array));//->Array
console.log(type(noNameFunc));//->function
console.log(type(nameFunc));//->function
console.log(type(nonNameFuncObj));//->noNameFunc
console.log(type(nameFuncObj));//->add

console.log(noNameFunc.getName());//->noNameFunc
console.log(nameFunc.getName());//->add
```

### 鸭子辩型

各种判断对象类的方法都有那样这样的问题，如果使用鸭子辩型编程思想则可以规避这样的问题，**不要关注对象的类是什么，而是关注对象能做什么**。这也可以称为**面向能力编程**。

示例，实现一个方法，用于判断一个对象是否具有某种能力：

```javascript
//各种判断对象类的方法都有那样这样的问题，如果使用鸭子辩型编程思想则可以规避这样的问题，**不要关注对象的类是什么，而是关注对象能做什么**。

/**
 * 如果对象o实现了参数中声明的函数就返回true，检测规则为：
 *      1. 如果参数是字符串，就判断o中是否有一个函数的名称与之匹配
 *      2. 如果参数是对象，就要求o生命了对象中的所有可枚举方法
 *      3. 如果参数是函数，就替换为函数的原型，然后按照对象的方式检测
 * @param o
 * @returns {boolean}
 */
function quacks(o /*, ... */) {
    for (var i = 1; i < arguments.length; i++) {  //注意从arguments的第2个参数遍历
        var arg = arguments[i];
        switch (typeof arg) {
            case 'string':       // 如果参数是字符串，就检测o中是否有这个函数
                if (typeof o[arg] !== "function") {
                    return false;
                }
                console.log(1);
                continue;
            case 'function':
                // 如果参数是函数就是用它的prototype属性代替，检测函数原型上的对象方法
                arg = arg.prototype;
            // 会进入下面的case语句，并再次执行arg的typeof值
            case 'object':       // object: check for matching methods
                for (var m in arg) { // 遍历该参数中的每一个属性
                    if (typeof arg[m] !== "function") {// 不是方法就跳过
                        continue;
                    }
                    if (typeof o[m] !== "function") {//如果对象o中没有对应的方法就返回false
                        return false;
                    }
                }
        }
    }
    //运行到这里则说明o对象实现了所有的函数
    return true;
}

//========================================================================
// 测试
//========================================================================
var obj = {
    m1: function () {

    }
};
var methods = {
    m1: function (x) {
        return x * x;
    }
};
var add = function (x, y) {
    return x + y;
};

var result1 = quacks(obj, add);
var result2 = quacks(obj, methods);
var result3 = quacks(obj, "add");

console.log(result1);//->true
console.log(result2);//->true
console.log(result3);//->false
```

---
## 9.6 JS中的面向对象技术

如何利用JavaScript的类进行面向对象编程。

### 用JavaScript实现set非重复无序集合

```javascript
//========================================================================
// 面向对象技术：实现Set集合。
//========================================================================

//定义构造函数
function Set() {
    this.values = {};     //该对象用于保持集合元素
    this.n = 0;           //集合有多少元素
    this.add.apply(this, arguments);  // 所有的参数作为变量加入
}

//定义add方法用于添加元素：添arguments的参数到这个set集合
Set.prototype.add = function () {
    for (var i = 0; i < arguments.length; i++) {
        var val = arguments[i];
        // 转换元素为一个String，作为key查找元素
        var str = Set._v2s(val);
        //判断一个对象是否有你给出名称的属性或对象。此方法无法检查该对象的原型链中是否具有该属性，该属性必须是对象本身的一个成员。
        if (!this.values.hasOwnProperty(str)) { // 如果不存在set中
            // 映射转换： string to value
            this.values[str] = val;
            // size自增
            this.n++;
        }
    }
    // 返回this用于支持链式调用
    return this;
};

//定义remove方法用以移除元素
Set.prototype.remove = function () {
    for (var i = 0; i < arguments.length; i++) {
        var str = Set._v2s(arguments[i]);
        if (this.values.hasOwnProperty(str)) {
            delete this.values[str];
            this.n--;
        }
    }
    return this;
};

//用于判断称号否存在某个元素
Set.prototype.contains = function (value) {
    return this.values.hasOwnProperty(Set._v2s(value));
};

// 返回Set的size
Set.prototype.size = function () {
    return this.n;
};

// 遍历set集合，在给定的上下文中调用传入的方法，参数为set中的每个元素
Set.prototype.foreach = function (f, context) {
    for (var s in this.values)                 // For each string in the set
        if (this.values.hasOwnProperty(s))    // Ignore inherited properties
            f.call(context, this.values[s]);  // Call f on the value
};

// 内部方法，用以将任何JavaScript值和对象用唯一的字符串对应起来
Set._v2s = function (val) {
    switch (val) {
        case undefined:
            return 'u';          // Special primitive
        case null:
            return 'n';          // values get single-letter
        case true:
            return 't';          // codes.
        case false:
            return 'f';
        default:
            switch (typeof val) {
                case 'number':
                    return '#' + val;    // Numbers get # prefix.
                case 'string':
                    return '"' + val;    // Strings get " prefix.
                default:
                    return '@' + objectId(val); // Objs and funcs get @
            }
    }

    // For any object, return a string. This function will return a different
    // string for different objects,
    // 对于任何一个对象，返回一个字符串，对于不同的对象这个函数将会返回一个不同的字符串。
    // 而对于同一个对象的多次调用，它总是返回相同的字符串
    // 为了做到这一点，给对象o创建了一个属性，在ES5中，这个属性是不可枚举且不可读的
    function objectId(o) {
        var prop = "|**objectid**|";   // 私有属性用于存储id
        if (!o.hasOwnProperty(prop))   // 如果对象没有id
            o[prop] = Set._v2s.next++; // 将下一个值分配给它
        return o[prop];                //返回对象的id
    }
};

Set._v2s.next = 100;    // 设置初始id 的值
```

### 用JavaScript实现枚举对象

```javascript
//========================================================================
// 面向对象技术：枚举的实现
//========================================================================

//这个函数创建一个新的枚举类型，实参对象表示类的毎个实例的名字和值
//返冋值是一个构造函数，它标识这个新类
//注意，这个构造函数也会抛出异常：不能使用它来创建该类型的新实例
//返回的构造函数包含名/值对的映射表
//包括由值组成的数组，以及一个foreach()迭代器函数
function inherit(p) {
    if (p === null) {
        throw  TypeError();
    }
    if (Object.create) {//如果由create方法
        return Object.create(p);
    }
    var t = typeof p;
    if (t !== "object" && t !== "function") {//必需是对象或者函数对象
        throw TypeError();
    }

    function F() {
    }

    F.prototype = p;
    new new F();
}

function enumeration(namesToValues) {

    // 这个虚拟的构造函数是返回值(创建一个函数)
    var enumeration = function () {
        throw "Can't Instantiate Enumerations";
    };

    // 枚举值继承这个对象
    var proto = enumeration.prototype = {
        constructor: enumeration, // 标识类型
        toString: function () {
            return this.name;
        }, // Return name
        valueOf: function () {
            return this.value;
        }, // Return value
        toJSON: function () {
            return this.name;
        }    // For serialization
    };

    enumeration.values = [];  // 用以存储枚举对象的数组

    // 创建新类型的实例
    for (name in namesToValues) {         // 对于每一个枚举变量
        var e = inherit(proto);          // 创建一个对象代表它
        e.name = name;                   // 给予其名字
        e.value = namesToValues[name];   //和值
        enumeration[name] = e;           // 将其设置构造函数的一个属性
        enumeration.values.push(e);      // 并且将其只存储到数组中
    }

    // 一个遍历方法
    enumeration.foreach = function (f, c) {
        for (var i = 0; i < this.values.length; i++) f.call(c, this.values[i]);
    };

    // 返回这个新的类型
    return enumeration;
}
```
enumeration函数返回的其实是一个函数对象，而枚举变量存储在函数对象的子类中，调用enumeration函数获取的枚举类只能当作普通对象来使用，这是很正常的，函数对象本身也是一个对象，这么做的目的就是实现枚举类只有有限个示例。

### 标准的转换方法

前面的类型转换涉及到一些重要的方法，有一些方法是由JavaScript做类型转换时自动调用的。不需要为每个类都定义这些方法，但是这些方法非常重要，不应忽略它们：

- `toString()`：这个方法的作用是返回一个可以表示这个对象的字符串。在希望使用字符串的地方用到对象的话（比如将对象用做属性名或使用“+”运算符来进行字符串连接运算），JavaScript会自动调用这个方法，如果没有实现这个方法，类会默认从`Object.prototype`中继承`toString()`方法，这个方法的运算结果是`“[object Object]"`,这个字符串用处不大，toString()方法应当返回一个可读的字符串，这样最终用户才能将这个输出值利用起来，然而有时候并不一定非荽如此，不管怎样，可以返回可读字符串的toStringO方法也会让程序调试变得更加轻松。
- `toLocaleString()`和`toString()`极为类似：`toLocaleString()`是以本地敏感性`(locale-sensitive)`的方式来将对象转换为字符串。默认情况下，对象所继承的`toLocaleString()`方法只是简单地调用`toString()`方法，有一些内置类型包含有用的`toLocaleString()`方法用以实际上返回本地化相关的字符串，如果需要为对象到字符串的转换定义toString()方法，那么同样需要定义toLocaleStringO方法用以处理本地化的对象到字符串的转换。
- `valueOf()`用来将对象转换为原始值：比如，当数学运算符（除了 “+" 运算符）和关系运算符作用于数字文本表示的对象时，会自动调用`valueOf()`方法，大多数对象都没有合适的原始值来表示它们，也没有定义这个方法。不过对于枚举类型valueOfO方法是非常重要的。
- `toJSON()`：这个发由`JSON.stringfy()`自动调用。

### 为JavaScript对象实现类似Java中的equals方法

>JavaScript的相等运算符比较对象时，比较的是引用而不是值。也就是说，给定两个对象引用，如果要看它们是否指向同一个对象，不是检査这两个对象是否具有相同的属性名和相同的属性值，而是直接比较这两个单独的对象是否相等，或者比较它们的顺序（就像“<”和“>”运算符进行的比较一样）。如果定义一个类，并且希望比较类的实例，应该定义合适的方法来执行比较操作，编程语言有很多用于对象比较的方法，将Java中的这些方法借用到JavaScript中是一个不错的主意。为了能让自定义类的实例具备比较的功能，定义一个名叫equals◦实例方法。这个方法只能接收一个实参，如果这个实参和调用此方法的对象相等的话则返回true。当然，这里所说的“相等”的含义是根据类的上下文来决定的。对于简单的类， 可以通过简单地比较它们的constructor属性来确保两个对象是相同类型，然后比较两 个对象的实例厲性以保证它们的值相等。——《JavaScript权威指南》

**注意，equals不是js中的标准，需要程序员手动调用。**

### 方法借用

>JavaScript中的方法没有什么特别：无非是一些简单的函数，赋值给了对象的属性，可以通过对象来调用它。一个函数可以同时陚值给两个属性，然后作为两个方法来调用它(方法复用)。

>多个类中的方法可以共用一个单独的函数。比如，Array类通常定义了一些内置方法， 如果定义了一个类，它的实例是类数组的对象，则可以从Array.prototype中将函数复制至所定义的类的原型对象中。如果以经典的面向对象语言的视角来看JavaScript的话， 把一个类的方法用到其他的类中的做法也称做“多重继承” （multiple inheritance), 然而，JavaScript并不是经典的面向对象语言，我更倾向于将这种方法重用更正式地称为 “方法借用” （borrowing)，不仅Array的方法可以借用，还可以自定义泛型方法（generic method)，其他类可以借用泛型方法：
```
Range.prototype.equals = generic.equals;
```
>注意，geneiic.equals()只会执行浅比较，因此这个方法并不适用于其实例太复杂的类。——《JavaScript权威指南》

```
var generic = {
    toString: function() {
        var s = '[';
        if (this.constructor && this.constructor.name)
            s += this.constructor.name + ": ";
        var n = 0;
        for(var name in this) {
            if (!this.hasOwnProperty(name)) continue;   // skip inherited props
            var value = this[name];
            if (typeof value === "function") continue;  // skip methods
            if (n++) s += ", ";
            s += name + '=' + value;
        }
        return s + ']';
    },

    equals: function(that) {
        if (that == null) return false;
        if (this.constructor !== that.constructor) return false;
        for(var name in this) {
            if (name === "|**objectid**|") continue;
            if (!this.hasOwnProperty(name)) continue;
            if (this[name] !== that[name]) return false;
        }
        return true;
    }
};
```

### 使用构造函数内部闭包实现私有对象的状态

面向对象的一个特性就是封装，对象的一些状态需要隐藏在对象内部，只能通过方法才能访问这些状态，JavaScript中没有类似Java中的权限修饰符，不过可以通过将变量或参数闭包在一个构造函数内来模拟实现私有字段。

```javascript
function Range(from, to) {
    //将from, to闭包在函数中，外部无法直接读取，而只能通过方法访问。
    this.from = function() { return from; };
    this.to = function() { return to; };
}

Range.prototype = {
    constructor: Range,
    includes: function(x) { return this.from() <= x && x <= this.to(); },
    foreach: function(f) {
        for(var x=Math.ceil(this.from()), max=this.to(); x <= max; x++) f(x);
    },
    toString: function() { return "(" + this.from() + "..." + this.to() + ")"; }
};
```

### 构造函数的重载和工厂方法

JavaScript中的方法的参数是可变的，对于通过方法，完全可以传入不同的参数，在函数内部对参数做判断，用以来实现函数的重载。

---
## 9.7 子类——形成继承层次

>在面向对象编程中，类B可以继承自另外一个类A。我们将A称为父类（superclass)，将B称为子类（subclass) , B的实例从A继承了所有的实例方法，类B可以定义自己的实例方法，有些方法可以重载类A中的同名方法，如果B的方法重载了A中的方法，B中的重载方法可能会调用A中的重载方法，这种做法称为“方法链” （method chaining), 同样，子类的构造函数B()有时需要调用父类的构造函数A(〉，这种做法称为“构造函数链” （constructor chaining)，子类还可以有子类，当涉及类的层次结构时，往往需要定义抽象类（abstract class)，抽象类中定义的方法没有实现。抽象类中的抽象方法是在抽象类的具体子类中实现的。 在JavaScript中创建子类的关键之处在于，采用合适的方法对原型对象进行初始化。如果类B继承自类A， B.prototype必须是A.prototype的后嗣。B的实例继承自B.prototype, 后者同样也继承自A.pnrtotype。——《JavaScript权威指南》


### 定义子类

如果O是B的子类，B是A的子类，那么O也一定继承了A的属性和方法，因此要确保B的原型对象继承自A的原型对象，可以通过之前定义的inherit方法：

```javascript
B.prototype = inherit(A.protetype);//子类派生自父类
B.protetype.constructor = B;//重载继承来的constructor
```
以上是JavaScript创建子类的关键，如果不这样做原型仅仅是一个普通对象。可以将这个关键过程封装为defineSubclass方法：

```javascript
//继承p
function inherit(p) {
    if (p == null) throw TypeError();
    if (Object.create)
        return Object.create(p);
    var t = typeof p;
    if (t !== "object" && t !== "function") throw TypeError();
    function f() {};
    f.prototype = p;
    return new f();
}

//赋值属性
function extend(o, p) {
    for(prop in p) {
        o[prop] = p[prop];
    }
    return o;
}

// 根据给定的超累创建一个子类
function defineSubclass(superclass,  // 超类的构造函数
                        constructor, // 新的子类的构造函数
                        methods,     // Instance methods: copied to prototype
                        statics)     // Class properties: copied to constructor
{
    // Set up the prototype object of the subclass
    constructor.prototype = inherit(superclass.prototype);
    constructor.prototype.constructor = constructor;

    // Copy the methods and statics as we would for a regular class
    if (methods) extend(constructor.prototype, methods);
    if (statics) extend(constructor, statics);

    // Return the class
    return constructor;
}

// 也可以通过父类的构造方法来做到这一点
Function.prototype.extend = function(constructor, methods, statics) {
    return defineSubclass(this, constructor, methods, statics);
};
```

### 构造函数和方法链——子类对方法的扩展而不是完全覆盖

在定义子类时，我们希望的是对父类的方法进行扩展，而不是完全的覆盖，为了做到这一点，构造函数和子类方法需要调用或链接到父类构造方法和父类方法。

```
//NonNullSet是Set的子类，不允许添加null和undefined元素。
function NonNullSet() {
    // 链接到父类
    // 作为普通函数来调用父类的构造方法来初始化
    Set.apply(this, arguments);
}

//让NonNullSet继承Set
NonNullSet.prototype = inherit(Set.prototype);
NonNullSet.prototype.constructor = NonNullSet;

// 重写add方法
NonNullSet.prototype.add = function() {
    for(var i = 0; i < arguments.length; i++)
        if (arguments[i] == null)
            throw new Error("Can't add null or undefined to a NonNullSet");

    // 链接父类方法以调用实际的添加方法
    return Set.prototype.add.apply(this, arguments);
};
```

可以继续优化上面的实现方式，定义一个类工厂函数，传入过滤函数(比如过滤null和undefined)，返回新的Set子类，可以让其变得更加通用化：
```javascript
//这个函数返回具体的Set，并对add方法调用传入的filter进行过滤
function filteredSetSubclass(superclass, filter) {
    var constructor = function() {          // 子类构造函数
        superclass.apply(this, arguments);  // 链接到父类
    };
    //让constructor继承superclass
    var proto = constructor.prototype = inherit(superclass.prototype);
    proto.constructor = constructor;
    //过滤
    proto.add = function() {
        // Apply the filter to all arguments before adding any
        for(var i = 0; i < arguments.length; i++) {
            var v = arguments[i];
            if (!filter(v)) throw("value " + v + " rejected by filter");
        }
        // 链接到超类的add方法
        superclass.prototype.add.apply(this, arguments);
    };
    return constructor;
}

//测试
var StringSet = filteredSetSubclass(Set, function(element){
                   return typeof element === "string";
                });
```
显然filteredSetSubclass工厂方法更加通用。

用一个函数将创建子类的代码包装起来，**这样就可以在构造函数和方法链中使用父类的参数**，而不是通过写死某个类的名字来使用它的参数，也就是说如果想修改父类，只需要改一处代码即可，而不必对每个用到父类类名的地方都做修改：
```javascript
var NonNullSet = (function () {
    var superclass = Set;
    //extend方法是defineSubclass调用哦
    return superclass.extend(
        function () {
            superclass.apply(this, arguments);
        },
        {
            add: function () {
                for (var i = 0; i < arguments.length; i++) {
                        if(arguments[i] ==null) {
                            throw new Error("can not add null or undefined")
                        }
                }
                return superclass.prototype.add.apply(this, arguments);
            }
        }
    );
}());
```

### 组合 VS 子类

上面，定义的集合可以根据特定的标准对集合成员做限制，而且使用了子类的技术来实现这种功能，所创建的自定义子类使用了特定的过滤函数来对集合中的成员做限制。父类和过滤函数的每个组合都需要创建一个新的类。然而还有另一种更好的方法来完成这种需求，即面向对象编程中一条广为人知的设计原则：**组合优于继承** 。这样，可以利用组合的原理定义一个新的集合实现，它“包装” 了另外一个集合对象，在将受限制的成员过滤掉之后会用到这个（包装的）集合对象。下面代码展示了其工作原理：

```javascript
//实现一个包装对象，它用于包装某个集合类，并根据filer对原有集合的元素进行过滤
var FilteredSet = Set.extend(

    function FilteredSet(set, filter) {  // The constructor
        this.set = set;
        this.filter = filter;
    },

    {  // 实例方法
        add: function() {
            // 如果有过滤器，调用它
            if (this.filter) {
                for(var i = 0; i < arguments.length; i++) {
                    var v = arguments[i];
                    if (!this.filter(v))
                        throw new Error("FilteredSet: value " + v +
                                        " rejected by filter");
                }
            }

            // 调用原有的add方法
            this.set.add.apply(this.set, arguments);
            return this;
        },

        //剩下的方法都保持不变
        remove: function() {
            this.set.remove.apply(this.set, arguments);
            return this;
        },
        contains: function(v) { return this.set.contains(v); },
        size: function() { return this.set.size(); },
        foreach: function(f,c) { this.set.foreach(f,c); }
    });

//对于不同过滤类型的集合，使用组合的好处是只需要创建单独的FilteredSet即可。
```

### 类的层级结构和抽象类

JavaScript中利用一些手段也可以模拟出抽象方法和抽象类，从而形成一个类的层次结构：

- 抽象方法是必须要时实现的，否则在运行时调用会抛出异常
- 抽象类是不可实例话的，否则调用它也会抛出异常

示例：
```javascript
//这个函数可以做任何抽象方法
function abstractmethod() {
    throw new Error("abstract method");
}

/*
 *  AbstractSet 定义了一个抽象方法：contains().
 */
function AbstractSet() {
    throw new Error("Can't instantiate abstract classes");
}
AbstractSet.prototype.contains = abstractmethod;

/*
 * NotSet是AbstractSet的非抽象子类，所有不在其他集合中的成员都在这个集合中，
 * 因为它是在其他集合不可写的条件下定义的，同时由于它的成员是无数个，因此它是被可枚举的，
 * 我们只能用它来及检测元素成员的归属情况。
 *
 * extend方法即defineSubsclass，用于创建一个子类。
 */
var NotSet = AbstractSet.extend(
    //子类构造函数
    function NotSet(set) {
        this.set = set;
    },
    //实例方法
    {
        contains: function (x) {
            return !this.set.contains(x);
        },
        toString: function (x) {
            return "~" + this.set.toString();
        },
        equals: function (that) {
            return that instanceof NotSet && this.set.equals(that.set);
        }
    }
);


/*
 * AbstractEnumerableSet是AbstractSet的抽象子类， 它定义了抽象方法 size() 和 foreach()，
 * 然后实现了非抽象方法：isEmpty(), toArray(), toString(), toLocaleString(), 和 equals()方法，
 * 子类实现了contains(), size(), and foreach()，这三个方法可以很轻易的调用这5个抽象方法
 */
var AbstractEnumerableSet = AbstractSet.extend(
    //子类构造函数
    function () {
        throw new Error("Can't instantiate abstract classes");
    },
    //实例方法
    {
        //抽象方法
        size: abstractmethod,
        foreach: abstractmethod,
        //非抽象方法
        isEmpty: function () {
            return this.size() == 0;
        },
        toString: function () {
            var s = "{", i = 0;
            this.foreach(function (v) {
                if (i++ > 0) s += ", ";
                s += v;
            });
            return s + "}";
        },
        toLocaleString: function () {
            var s = "{", i = 0;
            this.foreach(function (v) {
                if (i++ > 0) s += ", ";
                if (v == null) s += v; // null & undefined
                else s += v.toLocaleString(); // all others
            });
            return s + "}";
        },
        toArray: function () {
            var a = [];
            this.foreach(function (v) {
                a.push(v);
            });
            return a;
        },
        equals: function (that) {
            if (!(that instanceof AbstractEnumerableSet)) return false;
            // If they don't have the same size, they're not equal
            if (this.size() != that.size()) return false;
            // Now check whether every element in this is also in that.
            try {
                this.foreach(function (v) {
                    if (!that.contains(v)) throw false;
                });
                return true;  // All elements matched: sets are equal.
            } catch (x) {
                if (x === false) return false; // Sets are not equal
                throw x; // Some other exception occurred: rethrow it.
            }
        }
    });

/*
 * SingletonSet 是 AbstractEnumerableSet的子类，
 * SingletonSet只有一个元素且它是只读的。
 */
var SingletonSet = AbstractEnumerableSet.extend(
    //子类构造函数
    function SingletonSet(member) {
        this.member = member;
    },
    //实例方法
    {
        contains: function (x) {
            return x === this.member;
        },
        size: function () {
            return 1;
        },
        foreach: function (f, ctx) {
            f.call(ctx, this.member);
        }
    }
);


/*
 * AbstractWritableSet 是 AbstractEnumerableSet.
 * 它定义了抽象方法： add() and remove(),
 * 它实现了 union(), intersection(), and difference() 方法
 */
var AbstractWritableSet = AbstractEnumerableSet.extend(
    //子类构造函数
    function () {
        throw new Error("Can't instantiate abstract classes");
    },
    //对象方法
    {
        add: abstractmethod,
        remove: abstractmethod,
        union: function (that) {
            var self = this;
            that.foreach(function (v) {
                self.add(v);
            });
            return this;
        },
        intersection: function (that) {
            var self = this;
            this.foreach(function (v) {
                if (!that.contains(v)) self.remove(v);
            });
            return this;
        },
        difference: function (that) {
            var self = this;
            that.foreach(function (v) {
                self.remove(v);
            });
            return this;
        }
    });

/*
 *ArraySet是AbstractWritableSet的子类
 * 它数字的形式定义集合中的元素,
 * 对于contains方法它使用了数组的线性搜索，
 * 因为 contains()方法的复杂度是 O(n) 而不是 O(1),
 * 它非常适用于相对小型的集合，注意，这里的实现用到了ES5的数组方法indexOf和forEach()
 */
var ArraySet = AbstractWritableSet.extend(
    //子类构造函数
    function ArraySet() {
        this.values = [];
        this.add.apply(this, arguments);
    },
    //对象方法
    {
        contains: function (v) {
            return this.values.indexOf(v) != -1;
        },
        size: function () {
            return this.values.length;
        },
        foreach: function (f, c) {
            this.values.forEach(f, c);
        },
        add: function () {
            for (var i = 0; i < arguments.length; i++) {
                var arg = arguments[i];
                if (!this.contains(arg)) this.values.push(arg);
            }
            return this;
        },
        remove: function () {
            for (var i = 0; i < arguments.length; i++) {
                var p = this.values.indexOf(arguments[i]);
                if (p == -1) continue;
                this.values.splice(p, 1);
            }
            return this;
        }
    }
);
```




---
## 9.7 ES 5中的类

ES 5给属性添加了方法支持(setter，getter，可写性，可配置性，可枚举性)，还增加了对象的可扩展性限制，这些方法非常适合用于定义类，让类的定义变得更加健壮。

### 让类的属性不可枚举

在上面的利用JavaScript来实现Set集合的实例中，为了保证集合中对象的唯一性，需要给对象添加一个唯一的id标识，但是如果使用`for/in`来遍历对象，则这个标识id也会被遍历处理，利用ES 5中的defindProperty可以将这个id定义为不可标识的。

定义不可枚举的属性：

```javascript
// 用函数包装代码，我们可以在函数作用域内定义变量，而不会污染到全局上下文
(function () {
    // 定义一个不可枚举的属性objectId，它可以被所有对象集成到
    Object.defineProperty(Object.prototype, "objectId", {
        get: idGetter,       // id 的 getter方法
        enumerable: false,   // 不可枚举
        configurable: false  // 不能删除，不能配置
    });

    // 当读取对象的objectId时，该方法将被调用
    function idGetter() {             
        if (!(idprop in this)) {      
            if (!Object.isExtensible(this)) // 
                throw Error("Can't define id for nonextensible objects");
            Object.defineProperty(this, idprop, {         // 给对象分配唯一id
                value: nextid++,    // id
                writable: false,    // 只读
                enumerable: false,  // 不可枚举
                configurable: false // 不可配置
            });
        }
        return this[idprop];          // Now return the existing or new value
    };

    // 内部变量
    var idprop = "|**objectId**|";    // 假设不存在这个属性
    var nextid = 1;                   // 初始值

}());
```

### 定义不可变类

除了配置属性的可枚举性，ES 5还支持设置属性为只读属性，这对于创建不可变对象非常有帮助，只需要将属性设置为**只读和不可配置**即可，下面例子还展示了一个技巧，实现的构造函数还可以作为工厂方法，这样即使创建对象是不加上new关键字，也可以正确的创建一个新的对象：

```javascript
// 这个构造函数用于创建一个Range对象，即使不使用new关键字也可以正常工作
function Range(from, to) {
    // 属性描述符(只读，不可配置的)
    var props = {
        from: {value: from, enumerable: true, writable: false, configurable: false},
        to: {value: to, enumerable: true, writable: false, configurable: false}
    };

    if (this instanceof Range)                // 当被当成构造函数调用时执行：
        Object.defineProperties(this, props);//给对象定义属性
    else                                      // 当被当成工厂方法使用时执行：
        return Object.create(Range.prototype/*创建Range对象*/, props/*属性描述符集合*/);
}

// 其他属性，直接使用defineProperties定义，不需要管理它们的可枚举性
Object.defineProperties(Range.prototype, {
    includes: {
        value: function (x) {
            return this.from <= x && x <= this.to;
        }
    },
    foreach: {
        value: function (f) {
            for (var x = Math.ceil(this.from); x <= this.to; x++) f(x);
        }
    },
    toString: {
        value: function () {
            return "(" + this.from + "..." + this.to + ")";
        }
    }
});
```
Object.defineProperties和Object.create方法非常强大，但是上门代码的可读性可能并不那么好，另一种做法是先定义属性，然后再利用封装好的工具方法来修改已定义的属性的特性。

工具方法：
```javascript
//冻结对象的属性
function freezeProps(o) {
    var props = (arguments.length == 1)              // 如果只有一个参数
        ? Object.getOwnPropertyNames(o)              //  使用对象的所有属性
        : Array.prototype.splice.call(arguments, 1); // 否则使用传入的属性

    props.forEach(function (n) { // 遍历需要配置的属性，修改其为只读不可配置
        //忽略不可配置的属性
        if (!Object.getOwnPropertyDescriptor(o, n).configurable) return;
        Object.defineProperty(o, n, {writable: false, configurable: false});
    });
    return o; //返回
}

// 如果对象的属性是可配置的，让其变得不可枚举
function hideProps(o) {
    var props = (arguments.length == 1)
        ? Object.getOwnPropertyNames(o)
        : Array.prototype.splice.call(arguments, 1);
    props.forEach(function (n) {
        if (!Object.getOwnPropertyDescriptor(o, n).configurable) return;
        Object.defineProperty(o, n, {enumerable: false});
    });
    return o;
}
```

使用上面工具方法定义Range类：
```javascript
function Range(from, to) {
    this.from = from;
    this.to = to;
    freezeProps(this);
}

//重新定义Range.prototype
Range.prototype = hideProps(
    {
        constructor: Range,//构造函数为Range
        //增加下面方法
        includes: function(x) { return this.from <= x && x <= this.to; },
        foreach: function(f) {for(var x=Math.ceil(this.from);x<=this.to;x++) f(x);},
        toString: function() { return "(" + this.from + "..." + this.to + ")"; }
    }
);
```

### 封装对象状态

构造函数中的参数和变量可用用作它创建对象的私有状态，改方式在ES 3中有一个缺点：访问这些私有状态的存储器方法是可以替换的，**在ES5中可以通过定义setter和getter方法将状态变量更健壮的封装起来**。

```javascript
// 这个版本的Range是可变的，但是将端点变量进行了良好的封装，端点变量的大小顺序是固定的form < to
function Range(from, to) {
    // 验证参数
    if (from > to) throw new Error("Range: from must be <= to");

    // 定义存储器方法
    function getFrom() {  return from; }
    function getTo() {  return to; }
    function setFrom(f) { 
        if (f <= to) from = f;
        else throw new Error("Range: from must be <= to");
    }
    function setTo(t) { 
        if (t >= from) to = t;
        else throw new Error("Range: to must be >= from");
    }
    
    Object.defineProperties(this, {
        from: {get: getFrom, set: setFrom, enumerable:true, configurable:false},
        to: { get: getTo, set: setTo, enumerable:true, configurable:false }
    });
}

Range.prototype = hideProps({
    constructor: Range,
    includes: function(x) { return this.from <= x && x <= this.to; },
    foreach: function(f) {for(var x=Math.ceil(this.from);x<=this.to;x++) f(x);},
    toString: function() { return "(" + this.from + "..." + this.to + ")"; }
});
```

### 防止类的扩展

>通过给原型对象添加方法可以动态地对类进行扩展，这是JavaScript本身的特性。ECMAScript 5可以根据需要对此特性加以限制。Object.preventExtensions()可以将对象设置为不可扩展的，也就是说不能给对象添加任何新属性。 Object.seal()则更加强大，它除了能阻止用户给对象添加新属性，还能将当前已有的属性设置为不可配置的，这样就不能刪除这些属性了（**但不可配置的属性可以是可写的，也可以转换为只读属性**）。可以通过这样一句简单的代码来阻止对Object.prorotype 的扩展：`Object.seal(Object.prototype);`

>JavaScript的另外一个动态特性是“对象的方法可以随时替换”（或称为“monkey-patchn )
```
var original-Sortjnethod » Array.prototype.sort ；
Array.prototype.sort = function() {
    var start = new Date();
    original_sort_method.apply(this, arguments)；
    var end = new Date();
    console.log("Array sort took “ + (end - start) + “ milliseconds.");
}；
```
>可以通过将实例方法设置为只读来防止这类修改，一种方法就是使用上面代码所定义的freezeProps()工具函数。另外一种方法是使用Object.freeze()，它的功能和Object.seal()完全一样，它同样会把所有属性都设置为只读的和不可配置的.

>理解类的只读厲性的特性至关重要。**如果对象O继承了只读厲性P,那么给0.P的赋值操作将会失败，就不会给O创建新属性。如果你想重写一个继承来的只读属性，就必须使用 `Object.definePropertiy()`、`Object.defineProperties()`或`Object. create()`来创建这个新属性。**也就是说，如果将类的实例方法设S为只读的，那么重写它的子类的这些方法的难度会更大。
这种锁定原型对象的做法往往没有必要，但的确有一些场景是需要阻止对象的扩展的。但是对于上面的enumeration()类工厂函数，这个函数将枚举类型的每个实例都保存在构造函数对象的属性里，以及构造函数的values数组中，这些属性和数组是表示枚举类型实例的正式实例列表，是可以执行“冻结” (freezing)操作的，这样就不能给它添加新的实例，已有的实例也无法删除或修改。可以给enuraeration()函数添加几行简单的代码：
```
Object.freeze(enumeration.values)；
Object.freeze(enumeration);
```
>需要注意的是，通过在枚举类型中调用Object.freeze(),上门例子给对象添加的objectId属性之后也无法使用了，这个问题的解决办法是，在枚举类型被“冻结”之前读取一次它的objectId属性（这样它就被设置了objectI的属性了）。
>——《JavaScript权威指南》

### 子类和ES 5

利用ES 5来实现子类，`Object.create(null);` 用于创建一个没有原型的对象，可以对其直接使用`in`操作符，而不用使用`hasOwnProperty()`


下面示例用于创建AbstractWritableSet的子类：

```javascript
function StringSet() {
    this.set = Object.create(null); //一个没有原型的对象，用于存储元素
    this.n = 0;
    this.add.apply(this, arguments);
}

//使用Object.create可以继承父类的原型，而且可以定义单独调用的方法，
//因为没有指定属性的可写、可枚举、可配置性，因此这些属性的特性的默认值都是false，
//只读方法让这类难于被继承。
StringSet.prototype = Object.create(
   //原型
   AbstractWritableSet.prototype,
   //属性描述符
//start
   {
        constructor: { value: StringSet },//构造函数

        //实现对象方法
        contains: { value: function(x) { return x in this.set; } },
        size: { value: function(x) { return this.n; } },
        foreach: { value: function(f,c) { Object.keys(this.set).forEach(f,c); } },
        add: {
            value: function() {
                for(var i = 0; i < arguments.length; i++) {
                    if (!(arguments[i] in this.set)) {
                        this.set[arguments[i]] = true;
                        this.n++;
                    }
                }
                return this;
            }
        },
        remove: {
            value: function() {
                for(var i = 0; i < arguments.length; i++) {
                    if (arguments[i] in this.set) {
                        delete this.set[arguments[i]];
                        this.n--;
                    }
                }
                return this;
            }
        }
//end
    }
);
```

### 属性描述符

下面代码展示了属性描述符的相关操作，并封装到一个模块内：

```javascript
/*
给Object.prototype定义properties方法，这个方法返回一个表示调用它的对象上的属性名列表的对象(如果不带参数则表示该对象所有的属性)，返回的对象定义了四个有用的方法：
toString()
descriptors()
hide();
show();
*/
(function namespace() {  //私有作用域

     // properties方法
     function properties() {
         var names;  // 属性名数组
         if (arguments.length == 0)  //对象所有的属性
             names = Object.getOwnPropertyNames(this);
         else if (arguments.length == 1 && Array.isArray(arguments[0]))
             names = arguments[0];   //名字组成的数组，参数列表本身就是名字
         else
             names = Array.prototype.splice.call(arguments, 0);

         //返回新的Properties对象
         return new Properties(this, names);
     }

     //设置为不可枚举
     Object.defineProperty(Object.prototype, "properties", {
         value: properties,
         enumerable: false, writable: true, configurable: true
     });

     // 这个构造由properties方法调用，Properties表示一个对象的集合
     function Properties(o, names) {
         this.o = o;//属性所属对象
         this.names = names;//属性的名字集合
     }

     //将代表这些属性的对象设置为不可枚举的
     Properties.prototype.hide = function() {
         var o = this.o, hidden = { enumerable: false };
         this.names.forEach(function(n) {
                                if (o.hasOwnProperty(n))
                                    Object.defineProperty(o, n, hidden);
                            });
         return this;
     };

     //将这些属性设置为只读不可配置的
     Properties.prototype.freeze = function() {
         var o = this.o, frozen = { writable: false, configurable: false };
         this.names.forEach(function(n) {
                                if (o.hasOwnProperty(n))
                                    Object.defineProperty(o, n, frozen);
                            });
         return this;
     };

     // 返回一个对象，这个对象是名字到属性描述符的映射表
     // 使用右边代码来复制属性：Object.defineProperties(dest, src.properties().descriptors());
     Properties.prototype.descriptors = function() {
         var o = this.o, desc = {};
         this.names.forEach(function(n) {
                                if (!o.hasOwnProperty(n)) return;
                                desc[n] = Object.getOwnPropertyDescriptor(o,n);
                            });
         return desc;
     };

     // 返回一个格式良好的属性列表
     // 列表中包含属性、值和属性特征，使用permanent代表不可配置
     // readonly代表不可写，hidden表示不可枚举
     // 普通的可写、可枚举、可配置属性不包含特性列表
     Properties.prototype.toString = function() {
         var o = this.o; // Used in the nested function below
         var lines = this.names.map(nameToString);
         return "{\n  " + lines.join(",\n  ") + "\n}";

         function nameToString(n) {
             var s = "", desc = Object.getOwnPropertyDescriptor(o, n);
             if (!desc) return "nonexistent " + n + ": undefined";
             if (!desc.configurable) s += "permanent ";
             if ((desc.get && !desc.set) || !desc.writable) s += "readonly ";
             if (!desc.enumerable) s += "hidden ";
             if (desc.get || desc.set) s += "accessor " + n
             else s += n + ": " + ((typeof desc.value==="function")?"function"
                                                                   :desc.value);
             return s;
         }
     };

     //最后将原型对象的方法设置为不可枚举的
     Properties.prototype.properties().hide();
}()); // 立即调用
```


---
## 9.8 模块

>实现代码的重用，但类不是唯一的模块化代码的方式，一般来讲，模块是一个独立的JavaScript文件，模块文件可以包含一个类定义、一组相关的类、一个实用函数库或者是一些待执行的代码。只要以模块的形式编写代码，任何JavaScript代码段就可以当做一个模块JavaScript中并没有定义用以支持棋块的语言结构（但imports和exports的 确是JavaScript保留的关键字，因此JavaScript的未来版本可能会支持），这也意味着在 JavaScript中编写槟块化的代码更多的是遵循某一种编码约定， 很多JavaScript库和客户端编程框架都包含一些模块系统。比如，Dojo工具包和Google的Closure库定义了provide()和require()函数，用以声明和加栽模块，并且，[CommonJS](http://commonjs.org) 服务器端JavaScript标准规范创建了一个模块规范，后者同样使用require()函数。这种模块系统通常用来处理模块加栽和依赖性管理，这些内容已经超出本书的讨论范围，如果使用这些框架，则必须按照框架提供的模块编写约定来定义模块。

>本节仅对模块约定做一些简单的讨论。模块化的目标是支持大规模的程序开发，处理分散源中代码的组装，并且能让代码正确运行，哪怕包含了作者所不期望出现的棋块代码，也可以正确执行代码，为了做到这一点，不同的模块必须避免修改全局执行上下文，因此后续模块应当在它们所期望运行的原始（或接近原始）上下文中执行，这实际上意味若着模块应当尽可能少地定义全局标识，理想状态是，所有模块都不应当定义一个(全局标识)。

>作者这里的表述是围绕“糢块是一个重用的代码片役”这一观念的，不论是从代玛语法结构上解耦，还是将代码拆分至不同的文件中，只要用某种方法将代码“分离”，就认为是一个糢块，因此作者说任何代码都可以处理为一个糢块.
>这里的“原始上下文"是指调用模块时所在的上下文，可能处在一个很深的闭包当中，但这个模块的逻辑不应该影响到其他的上下文特别是全局上下文——《JavaScript权威指南》

总而言之，模块之间应当避免命名冲突，即全局变量的污染，下面是书中提供的两种方式：

### 用做命名空间的对象

在模块创建过操中避免污染全局变量的一种方法是使用对象作为命名空间。它将函数和值作为命名空间对象属性存储起来，可以通过全局变量引用，而不是定义全局函数和变量。

### 作为私有命名空间的函数

模块对外导出一些公用API，这些API是提供给其他程序员使用的，它包括`函数、类、厲 性和方法`，但模块的实现往往需要一些额外的辅助函数和方法，这些函数和方法并不需要在模块外部可见。模块作者不希望用户在某时刻调用这些辅助函数，因此这个方法最好在类的外部是不可访问的。 可以通过将模块定义在某个函数的内部来实现。在一个函数中定义的变量和函数都属于函数的局部成员，在函数的外部是不可见的，实际上，可以将这个函数作用域用做模块的私有命名空间（有时称为“模块函数”）。

示例：

```javascript
// 生命一个全局的变量Set，Set是它右边函数的返回值
// 下面圆括号中定义的方法在定义后将被立即调用, 并且它是函数的返回值，不是被分配的函数本身
// 这不是函数表达式，不是语句，所以不会创建一个全局的变量
var Set = (function invocation() {

    function Set() {  // 构造函数是本地变量
        this.values = {};     // set的属性集
        this.n = 0;           // 数量
        this.add.apply(this, arguments);  // 默认的参数都被添加
    }

    // 定义对象方法
    Set.prototype.contains = function(value) {
        return this.values.hasOwnProperty(v2s(value));
    };
    Set.prototype.size = function() { return this.n; };
    Set.prototype.add = function() { /* ... */ };
    Set.prototype.remove = function() { /* ... */ };
    Set.prototype.foreach = function(f, context) { /* ... */ };

    // 下面这些是赋值方法，不会暴露给使用者
    function v2s(val) { /* ... */ }
    function objectId(o) { /* ... */ }
    var nextId = 1;

    // 暴露公共的API：Set构造函数
    return Set;

}());//顶以后立即调用
```

这里使用了立即执行的匿名函数，这在JavaScript中是一种惯用法。如果想让代码在一个私有命名空间中运行，只须给这段代码加上前缀`(function( { `和后缀 `} ())”`。开始的左圆括号确保这是一个函数表达式，而不是函数定义语句，因此可以给该前缀添加一个函数名来让代码变得更加淸晰。在上门例子中使用了名字“invocation” ,用以强调这个函数应当在定义之后立即执行，名字"namespace”也可以用来强调这个函数被用做命名空间。

一旦将模块代码封装进一个函数，就需要一些方法导出其公用API,以便在検块函数的外部调用它们。上面例子中，模块函数返回构造函数，这个构造函数随后赋值给一个全局变量。将值返回已经清楚地表明API已经导出在函数作用域之外。如果模块API包含多个单元，则它可以返回命名空间对象。对于sets棋块来说，可以将代码写成这样：
```
//创建一个全局变量用来存放集合相关的模块
var collections;

if (!collections) {
   collections = {};
}
//定义sets棋块
collections.sets = (function namespace() {
           //这里省略很多代码
           function Set(){
              ......
           }
           return Set;
}());
```

另外一种类似的技术是将模块函数当做构造函数，通过new来调用，**通过将它们赋值给this来将其导出**，如：
```javascript
var collections;
if (!collections) {
   collections = {};
}
collections.sets = (new function namespace() {
   //……这里省略很多代码……
   //将API导出至this对象
   this.AbstractSet = AbstractSetj;
   thls.NotSet = NotSet;
   //注意，这里没有返回值
}());
```

作为一种替代方案，如果已经定义了全局命名空间对象，这个模块函数可以直接设置那个对象的属性，不用返回任何内容：

```javascript
var collections;
if (!collections){
   collections = {};
}
collections.sets = (function namespace() {
   //……这里样略很多代码……
   //将共用API导出到上面创雄的命名空间对象上
   collections.sets.AbstractSet “ AbstractSet;
   collections.sets.NotSet - NotSet;
   //导出的操作已经执行了，这里不需要return语句了
}());
```
有些框架实现了模块加载功能，其中包括其他一些导出模块API的方法。比如，使用provides()函数来注册其API，提供exports对象用以存储模块API。 由于 JavaScript 目前还不具备模块管理的能力，因此应当根据所使用的框架和工具包来选择合适的模块创建和导出API的方式。



---
# 10 正则表达式

正则表达式（regular expression)是一个描述字符棋式的对象，JavaScript的RegExp类表示正则表达式，**String和RegExp都定义了方法**，后者使用正则表达式进行强大的模式匹配和文本检索与替换功能。同时在动态的创建正则表达式时，只能使用RegExp。在ES3中，相同的正则表达式字符串总是返回同一个RegExp对象，而在ES5中，每次都会创建一个新的RegExp对象。

```javascript
//定义正则方式一：对象
var rg1 = new RegExp("^\\d+$");

//定义正则方式二：/符号，不需要字符串的转义
var rg2 = /^\d+$/;

var s = "1234";
var r1 = s.match(rg1);//String.match(正则)。返回的是匹配的结果（不是true或false），如果不匹配返回null

var r2 = rg2.test(s);//正则对象的test(字符串)，匹配返回的是true，否则是false。
console.log(r1);//->[ '1234', index: 0, input: '1234' ]
console.log(r2);//->true
```

---
## 10.1 String 对象的正则相关方法

- match
- search
- replace

---
## 10.2 RegExp 对象

### RegExp 的属性

每个RegExp对象都包含5个属性。

- 属性source是一个只读的字符串，包含正则表达式的 文本。
- 属性global是一个只读的布尔值，用以说明这个正则表达式是否带有條饰符`g`。
- 厲性ignoreCase也是一个只读的布尔值，用以说明正则表达式是否带有修饰符`i`。
- 属性multiline是一个只读的布尔值，用以说明正则表达式是否带有修饰符`m`。
- lastIndex是一个可读/写的整数。如果匹配模式带有`g`辦饰符，这个属性存储在整个字符串中下一次检索的开始位置,这个属性会被`exec()`和`test()`方法用到。

### RegExp 的方法

- exec
- test