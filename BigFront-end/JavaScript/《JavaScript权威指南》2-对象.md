# 6 对象

---
## 6.1 认识对象

对象的定义：**是一种JavaScript的基本数据类型，是一种复合值，由很多值聚合在一起，可以通过名字来访问这些值，对象可以看作是属性的无序集合。**，JavaScript的对象是**动态**的，可以随意添加和删除属性。

对象最常见的用法是**创建、设置、查找、删除、监测、枚举**它的属性。

属性包括**名和值**，不能同时存在同名的属性，除了名和值，属性还有与之关联的**“属性特征”**。

对象的"属性特征"包括：

  - 可写：是否可以设置该属性
  - 可枚举：是否可以通过for/in循环返回该属性
  - 可配置：是否可以删除或者修改该属性

对象除了包含属性之外，还包括三个相关对象特性：

  - 对象的原型(propotype)：指向另一个对象，本对象的属性继承自它的原型对象
  - 对象的类(class)：**标识对象的类型的字符串**
  - 对象的扩展标记(extensible flag)：指明了是否可以向该对象添加新的属性

总结对三类JavaScript对象和两类属性作区分：

- 内置对象(`native object`)：由ECMAScrpit规范定义了的对象或类
- 宿主对象(`host object`)：由JavaScript解释器所嵌入的宿主环境
- 自定义对象(`user-defined object`)：JavaScript运行中创建的对象
- 自有属性(`own property`)：直接在对象中定义的属性
- 继承属性(`inherited property`)：在对象的原型对象中定义的属性


### 创建对象

创建对象的方式：

- 对象直接量
- 关键字**new**
- ES5的`Object.create()`静态方法。

>注意：使用对象直接量方式，最后一个属性的逗号将被忽略，但是在IE中却会报错。

```javascript
//对象直接量
var empty = {};
var point = {x: 1, y: 2};

//通过new创建对象
var obj = new Object();
var reg = new RegExp("js");

//通过Object的create方法创建新对象
//create方法接受两个参数，第一个参数为原型(通过__proto__获取)，第二个参数为可选参数，用以描述对象的属性，即属性描述符的集合。create方法是一个静态函数
var obj1 = Object.create({x: 1, y: 2});//obj1继承了x和y属性

var obj2 = Object.create(null);//可以传入null值，创建一个没有原型的对象，没有继承任何属性和方法，甚至是toString()

var obj3 = Object.create(Object.prototype);//创建普通对象，等效于{}

//属性描述符，参考下面章节
var props = {
   from: {value: 0, enumerable: true, writable: false, configurable: false},
   to: {value: 100, enumerable: true, writable: false, configurable: false}
};
Object.create(Object.prototype/*创建Range对象*/, props/*属性描述符集合*/);
```

在ES3中封装工具方法inherit方法(并不能完全代替ES5中的create方法)：

```javascript
//在ES3中封装工具方法inherit，inherit返回继承一个继承自对象p的新对象，inherit可以防止库函数无意间(非恶意的)修改那些不受你控制的对象，
//不直接传入对象，而是将它的继承对象传入，因为当函数读取集成对象的属性时，是读取集成过来的值，
//如果给继承对象的属性赋值，则这些属性只会影响这个继承对象自身，而不是原始对象
function inherit(p) {
    if (p == null) {
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
```


### 原型

原型：每一个js对象(null除外)都和另一js对象关联，这里的另一个对象就是原型。每一个对象都从原型那里继承属性。

- 通过对象直接量创建的对象都具有相同的原型对象，并且可以通过Object.prototype获得该原型对象的引用。
- 通过new关键字和构造函数调用创建的对象的原型就是构造函数的prototype属性值。
- 没有原型的对象位数不多，Object.prototype就是其中之一。可以认为Object.prototype是原型链中的顶层
- 对象之间的继承关系构成了原型链

---
## 6.2 属性的查询和设置

- 通过`.`或者`[]`访问对象的属性：`object.property`；`object["property"]`
- 作为关联数组的对象：通过`object["property"]`方式来访问属性，看起来像数组，但是key是字符串而不是数值，这种数组称为关联数组，或者交散列、映射、字典。**JavaScript对象都是关联数组**
- `object.property`和`object["property"]`方式的区别：.后面跟着的标识符必需是确定的，而[]里的字符串可以是动态的。


### 继承

- 对象的属性包括**自有属性和继承属性**，当访问对象的属性时，先看对象本身是否有该属性，没有才会查找原型上是否有该属性
- 对象的原型属性构成了一个**链**，通过这个链可以实现属性的继承
- 如果一个属性是可以赋值的(非只读属性)，那么它总是在原始对象上创建或者对已有的属性进行赋值，而不会修改原型链，**只有在查询的时候才能体会到继承的存在**，而设置属性与继承无关。
- **继承来的属性是可以被覆盖的。**

### 属性访问错误

- 给null和undefined设置属性回报类型异常
- 一些只读属性不能重新赋值，但是操作只会失败不会报错
- 有一些对象不允许新增属性，但是操作只会失败不会报错

赋值属性要么失败，要么创建一个属性，要么在原始对象中设置属性，有一个例外，如果对象o继承了属性x，而这个属性是一个具有setter方法的accessor属性，给对象o设置x属性时，那么这时将调用此方法而不是给o创建一个属性x

给对象o设置属性p会失败的场景：

- p属性是只读的，不能重新赋值(可配合的只读属性除外)
- p是继承来的只读属性，此时p不能被覆盖
- o中不存在属性p，o没有setter方法继承属性p，且o的可扩展性为false


----
## 6.3 删除属性

delete表达式可用于删除属性

- `delete`只是断开属性和宿主的关系，而不会去操作属性中的属性
- `delete`只能删除自有属性，不能删除继承属性
- `delete`表达式删除成功或者没有副作用时返回true，否则返回false
- `delete`不能删除可配置性为false的属性
- **严格模式下，删除全局对象的属性必需加上this引用，否则报错**

----
## 6.4 检测属性

判断某个属性是否在对象中可用的方法：

- `in运算符`
- 对象方法`hasOwnProperty()`
- 对象方法`propertyIsEnumerable()`

方法说明：

- `in`用来检查对象的可枚举属性
- `hasOwnProperty()`用来检查对象的自有属性
- `propertyIsEnumerable()`用来检查对象的可枚举自有属性
- 一般js代码创建的属性都是可枚举的，ES5中可以创建不可枚举的属性
- `!== undefined`也可用来判断对象中是否存在某个值不是undefined的属性

```javascript
//检查属性
var o = {x: 1};
console.log("x" in o);//->true
console.log("y" in o);//->false
console.log("toString" in o);//->true

console.log(o.hasOwnProperty('x'));//->true
console.log(o.hasOwnProperty('y'));//->false
console.log(o.hasOwnProperty('toString'));//->false

var o2 = Object.create({y: 3});
o2.x = 1;
console.log(o2.propertyIsEnumerable("x"));//true
console.log(o2.propertyIsEnumerable("y"));//false，y是继承来的
console.log(Object.prototype.propertyIsEnumerable("toString"));//false，toString不可枚举
```

----
## 6.5 枚举属性

1：`for/in`循环可以用来遍历对象中所有的可枚举属性名，包括**自有属性和继承属性，包括可枚举的方法(方法本身也是对象)**

2： ES5中增加了两个方法用以枚举对象属性
  - `Object.keys(obj)`静态方法返回一个数组，数组由对象的所有可枚举自有属性名称组成
  - `Object.getOwnPropertyNames(obj)`静态方法，和keys方法类似，只是它返回对象的所有自有属性名称，包括不可枚举的。

```javascript
// 枚举属性方法

//复制p中可枚举属性到o中
function extend(o, p) {
    for (var prop in  p) {
        o[prop] = p[prop];
    }
    return o;
}
console.log(extend({}, {x: 3}));

//复制p中可枚举属性到o中，已有属性不覆盖
function merge(o, p) {
    for (var prop in  p) {
        if (o.hasOwnProperty(prop)) {
            continue;
        }
        o[prop] = p[prop];
    }
    return o;
}

//删除o中属性在p中不存在的同名属性，就删除o中的该属性
function restrict(o, p) {
    for (var prop in  o) {
        if (!(prop in p)) {
            delete o[prop];
        }
    }
    return o;
}

//如果属性在o和p中同时存在，删除o中的该属性
function subtract(o, p) {
    for (var prop in  o) {
        if (prop in p) {
            delete o[prop];
        }
    }
    return o;
}

//返回新对象，新对象同时拥有p和o中的属性,有同名属性时使用p的
function union(o, p) {
    return extend(extend({}, o), p);
}

//返回新对象，新对象有用同时存在o和p中的属性，同名属性时，使用p 的
function intersection(o, p) {
    return restrict(extend({}, o), p);
}

//返回所有可枚举的自有属性的名称组成的数组
function keys(o) {
    if (typeof  o !== "object") {
        throw new TypeError;
    }
    var result = [];
    for (var prop in o) {
        if (o.hasOwnProperty(prop)) {
            result.push(prop)
        }
    }
    return result;
}
console.log(keys({x: 3, y: 4}));
```

----
## 6.6 属性的setter和getter

对象是由名字、值和一组特性构成的，在ES5中，属性可以用一个或两个方法代替，这两个方法就是setter和getter，由setter和getter定义的属性称为**存储器属性(accessor property)**。

- 当查询属性时，getter方法被调用，当设置属性时，setter方法被调用。
- 如果只有getter方法，说明属性是只读属性，如果只有setter方法，说明属性是只写属性。
- 存储器属性是可以被继承的

```
var p = {
    //普通属性
    x: 1.0,
    y: 1.0,
    //存储器属性
    get r() {
        return Math.sqrt(this.x * this.x + this.y * this.y);
    },
    set r(newValue) {
        //this表示指向这个点的对象
        var oldValue = Math.sqrt(this.x * this.x + this.y * this.y);
        var ratio = newValue / oldValue;
        this.x *= ratio;
        this.y *= ratio;
    }
};

//继承存储器属性
var q = Object.create(p);
q.x = 3;
q.y = 4;
console.log(q.r);//->5
```

>在ECMAScript 5标准被采纳之前，大多数JavaScript的实现（IE浏览器除外）已经可以支持对象直接量语法中的get和set写法。这些实现提供了非标准的老式API用来査询和设置getter和setter,这些API由4个方法组成，所有对象都拥有这些方法，`__lookupGetter__()`和`__lookupSetter__()`用以返回一个命名属性的 `getter` 和 `setter` 方法，`__defineGette__`(）和 `__defineSetter__()`用以定义getter和setter，这两个函数的第一个参数是属性名字，第二个参数是getter和setter方法，这4个方法都是以两条下划线作前缀，两条下划线作后缀，以表明它们是非标准的方法。

## 6.7 属性的特性

除了包含名字和值外，属性还包括一些标识它们**可写、可枚举、可配置的特性**。在ES3中无法设置只写属性，所有ES3中程序创建的属性都是可配置，可写，可枚举的。在ES5中规范了查询和设置这些特性的API：

- 可以通过API给原型对象添加方法，并将它们设置为不可枚举的。
- 可以通过API给对象定义不能修改或者删除的属性，借此锁定这个对象。

可以将数据属性的值看作是属性的特性。因此可以认为一个属性包括一个名字和四个属性。这四个属性是：

- 值(value)
- 可写性(writable)
- 可枚举性(enumerable)
- 可配置性(configurable)

可以将存储器属性的setter和getter看作是属性的特性，存储器属性不具有值特性和可写特性，因此存储器属性的四个特性是：

- 取(get)
- 写(set)
- 可枚举性(enumerable)
- 可配置性(configurable)

为了实现属性特性的查询和操作，ES5中定义了一个名为**属性描述符(property descriptor)**的对象。

数据属性的属性描述符具有：`value、witeable、enumerable、configurable`四个值，其中`witeable、enumerable、configurable`都是布尔类型，存储器类型的属性描述符具有：`get、set、enumerable、configurable`四个值，其他get和set都是函数值。

通过`Object的getOwnPropertyDescriptor()`静态方法可以获取对象特定属性的属性描述符。getOwnPropertyDescriptor方法只能获取自有属性的描述符，想要获得继承属性的特性，需要遍历原型链。

### 设置属性的特性

想要设置(修改)属性的特性，或者想让新建属性具有某种特性，可以使用`Object.defineProperty()`静态方法。

属性描述符：

```javascript
//->{ value: 1, writable: true, enumerable: true, configurable: true }
var pd = Object.getOwnPropertyDescriptor({x: 1}, 'x');

var accessorObj = {
    get value() {
        return 1;
    },
    set value(value) {

    }
};
//->{ get: [Function: get value], set: [Function: set value], enumerable: true, configurable: true }
var accessorPd = Object.getOwnPropertyDescriptor(accessorObj, 'value');
```

defineProperty：

```javascript
var o = {};
Object.defineProperty(o, "x", {
    value: 1,
    writable: true,
    enumerable: true,
    configurable: true
});
console.log(o.x);//->1
console.log(Object.keys(o));//->[ 'x' ]

//现在对属性x进行修改，让其变为只读
Object.defineProperty(o, "x", {
    writable: false
});
o.x = 4;//操作失败，不报错
console.log(o.x);//->1

//由于x依然是可以配置的，可以通过下面方法改变x的值
Object.defineProperty(o, "x", {
    value: 4
});
console.log(o.x);//->4

//现在将x从数据属性改为存储器属性
Object.defineProperty(o, "x", {
    get: function () {
        return 0;
    }
});
console.log(o.x);//->0
```

defineProperty方法传入的属性描述符，不需要传入所有的值，默认的特性值是false或者undefined。并且该方法不能修改继承属性。同时要修改多个属性可以使用`defineProperties`方法。**对于那些不允许创建或者修改的属性来说，使用defineProperty方法就会抛出类型错误**，比如给一个不可扩展的对象新增属性。

defineProperty的规则：

- 对象不可扩展，可写修改已有属性，不可新增属性
- 如果属性不可配置，不能修改其可配置性和可枚举性
- 如果存储器属性不可配置，不能修改其getter和setter方法，亦不能将其改为数据属性
- 如果数据属性不可配置，不能将其改为存储器属性
- 如果数据属性不可配置，则不能将其可写性由false改为true，但可以从true改为false
- 如果数据属性不可配置且不可写，则不能修改其值，但是可配置属性但不可写属性的值是可以修改的(需要通过defineProperty方法修改)

给顶级原型添加一个健壮的extend方法：

```javascript
//extend方法用于复制对象的属性到传入的对象
Object.defineProperty(Object.prototype, "extend", {
    writable: true,
    enumerable: false,
    configurable: true,
    value: function (o) {
        var names = Object.getOwnPropertyNames(o);//得到所有自有属性的名称，包括不可枚举属性
        for (var i = 0; i < names.length; i++) {
            if (names[i] in this) {//this指向方法调用者，如果已经存在这些属性，跳过
                continue;
            }
            var desc = Object.getOwnPropertyDescriptor(o, names[i]);
            Object.defineProperty(this, names[i], desc);
        }
    }
});
```

---
## 6.8 对象的三个属性

每个对象都有与之关联的**原型、类、可扩展性**。

### 原型属性

对象的原型属性是用来继承属性的。

- 通过对象直接量创建的属性的原型是`Object.prototype`
- 通过new创建的对象使用构造函数的prototype属性作为它的原型
- 通过`Object.create()`创建的对象使用该方法的第一次参数作为其原型，这个参数可以为null。

ES5中可以使用`Object.getPrototypeOf()`静态方法查询对象的原型。

ES3中没有这样的方法。可以使用`obj.constructor.prototype`属性来检测一个对象的原型，通过new表达式创建的对象，通常继承一个constructors属性，这个属性指向创建这个对象的构造函数，但这种查询原型的方法并不可靠。**注意**：通过对象直接量或者`Object.create()`方法创建的对象的包含一个constructor属性，这个属性指代Object()构造函数，所以`constructor.prototype`才是对象直接量的真正的原型，但是对于`Object.create()`创建的对象来说往往不是这样的。

应该使用对象`isPrototypeOf()`方法判断一个对象是不是另一个对象的原型。比如`p.isPrototypeOf(o)`用来检查p是否是o的原型。

浏览器JS引擎中提供了`__proto__`属性用与获取对象的原型，而且由于很多浏览器都支持，它已经是ES6标准了。

```javascript
var p = {x: 1};
var o = {x: 2};
p.isPrototypeOf(o);//->false 查询属性

var newP = Object.create(p);
p.isPrototypeOf(newP);//-> true
var newPConstructor = newP.constructor;//->[Function: Object]
var pConstructor = p.constructor;//->[Function: Object]

// 使用create方法创建对象时，不能使用此方式获取原型属性，这里得到的是{}。应该使用`isPrototypeOf()`方法判断一个对象是不是另一个对象的原型。
Object.getPrototypeOf(newP);//->{x:1}
var fakePrototype = newP.constructor.prototype;//->{}
```


### 类属性

对象的类属性是一个字符串。用以表示对象的类型信息。ES5和ES3中都没有用于查询对象类属型的方法，只能通过间接的toString方法来查询。

```javascript
o.toString(); // ->[object class],object后面的字符串就是对象的类型信息。
```

但是很多对象重写了toString方法，需要通过以下方式查询。

```javascript
var a = {x: 123};
console.log(a);//->{x:123};，log会将对象的内容打印出来，但是并不是调用对象的头String方法
var aToString = a.toString();//->[object Object]
var aClass = a.toString().slice(8, -1);//-> Object

//定义方法获取对象的class属性
function classOf(o) {
    if(o === null) {
        return "Null";
    }
    if(o===undefined) {
        return "Undefined";
    }
    //间接的调用Function.call()方法
    return Object.prototype.toString.call(o).slice(8, -1);
}
```

### 可扩展性

表示是否可以给一个类新增属性。所有的内置对象和自定义对象默认都是可扩展的，宿主对象的可扩展性由js引擎决定。

- `Object.isExtensible` 判断是否可扩展
- `Object.preventExtensions` 变为不可扩展，不可逆
- `Object.seal` 封闭对象，把对象变为不可扩展，并且所有属性改为不可配置
- `Object.isSealed`
- `Object.freeze` 冻结对象，把对象变为不可扩展，并且所有属性改为不可配置且位置为只读，但是setter不受影响
- `Object.isFrozen`

```javascript
var c = {};
Object.isExtensible(c);//->true，判断是否可扩展
Object.preventExtensions(c);//锁定对象，把对象变为不可扩展

var d = {x: 1, y: 2};
Object.seal(d);//封闭对象，把对象变为不可扩展，并且所有属性改为不可配置
Object.isSealed(d);//->true，是否是封闭对象

var e = {x: 32};
Object.freeze(e);//冻结对象，把对象变为不可扩展，并且所有属性改为不可配置且位置为只读，但是setter不受影响
Object.isFrozen(e);//-> true

//创建一个封闭对象，第2个参数为属性描述对象
var f = Object.create(Object.freeze({x: 1}), {
    y: {value: 3, writable: true}
});
console.log(f);//->{}，冻结对象属性不可枚举
console.log(f.y);//->3
```

---
## 6.9 对象序列化

对象序列化将对象的状态转为字符串。反序列化可以将字符串转为对象。

ES5中`JSON.stringify()`方法用于将对象转换为json(JavaScript Object Notation)字符串,`JSON.parse()`方法用于将字符串反序列化为对象。

JSON语法是js语法的子集，但是其并不能表示js里所有的值。其支持**对象、数组、字符串、无穷大数、true、false和null**。并且可以被序列化和反序列化。NaN，Infinity和-Infinity序列化的结果是null,日期对象序列化的结果是ISO袼式的日期字符串，JS0N.parse()依然保留它们的字符串形态，而不会将它们还原日期对象，函数、RegExp、Error对象和undefined值不能序列化和还原，JS0N.stringify()只能序列化对象可枚举的属性，对应一个个不能序列化的属性性来说，在序列化后的输出字符串中会将这个属性省略掉，JSON.stringifyO和JSON.parseOf可以接收第二个对选参数，通过传入需要序列化或还原的属性列表来定制自定义的序列化和还原操作。


---
## 6.10 对象的方法

几乎所有的对象都从Object.prototype继承属性，这些继承的属性主要都是方法，这些方法包括：

- `hasOwnProperty()`：用来判断一个对象是否有你给出名称的属性或对象。不过需要注意的是，此方法无法检查该对象的原型链中是否具有该属性，该属性必须是对象本身的一个成员。
- `propertyIsEnumerable()`：属性是否可枚举的
- `isPrototypeOf()`：原型链判断
- `toString()`：将该对象的原始值以字符串形式返回
- `toLocalString()`：返回表示这个对象的本地化的字符串
- `valueOf()`：返回最适合该对象类型的原始值

Object.prototype中没有定义toJSON()这个方法，但是对于需要序列化的对象来说。JSON.stringify会调用toJSON()方法。如果待序列化的对象存在这个方法，则调用它。返回的就是序列化的结果，而不是原始对象。

Object的静态函数：

- `create()`：创建对象
- `getPrototypeOf()`：返回原型对象
- `defineProperty()`：定义属性
- `getOwnPropertyNames()`：获取所有的自有属性名称，包括不可枚举的
- `keys()`：返回一个数组，数组由对象的所有可枚举自有属性名称组成
- `getOwnPropertyDescriptor()`：获取属性描述符
- `isExtensible()`：是否扩张
- `preventExtensions()`：设置为不可扩展，不可逆
- `seal()`：封闭对象
- `isSealed()`：是否是封闭对象
- `freeze()`：冻结对象
- `isFrozen()`：是否是冻结对象


----
## 6.11 关于constructor属性

constructor属性有作用，搜了一下修改资料，有以下回答：

1： 为了将实例的构造器的原型对象暴露出来, 比如你写了一个插件,别人得到的都是你实例化后的对象, 如果别人想扩展下对象，就可以用 `instance.constructor.prototype` 去修改或扩展原型对象


2：也有人说**只是JavaScript语言设计的历史遗留物**
 - constructor属性不影响任何JavaScript的内部属性。instanceof检测对象的原型链，通常你是无法修改的（不过某些引擎通过私有的__proto__属性暴露出来）。
 - constructor其实没有什么用处，只是JavaScript语言设计的历史遗留物。由于constructor属性是可以变更的，所以未必真的指向对象的构造函数，只是一个提示。不过，**从编程习惯上，我们应该尽量让对象的constructor指向其构造函数，以维持这个惯例。**

参考：

- [JavaScript 中对象的 constructor 属性的作用是什么？](https://www.zhihu.com/question/19951896)