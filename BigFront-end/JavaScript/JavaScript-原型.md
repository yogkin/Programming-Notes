# JavaScript原型

![原型链图](index_files/1527145567818prototype_map.png "原型链图")

## 1 普通对象和函数对象

JavaScript中，万物皆对象，但对象也是有区别的。分为**普通对象和函数对象**，Object 、Function是JS自带的函数对象。

下面代码o1 o2 o3为普通对象，f1 f2 f3为函数对象。怎么区分?

其实很简单，**凡是通过 new Function() 创建的对象都是函数对象，其他的都是普通对象。**f1、f2归根结底都是通过`new Function()`的方式进行创建的。Function Object 也都是通过 `new Function()`创建的。

```javascript
var o1 = {};
var o2 = new Object();
var o3 = new f1();

function f1() {
}
var f2 = function () {
};
var f3 = new Function('str', 'console.log(str)');

var type = typeof Object; //->function
var type = typeof Function; //->function
var type = typeof f1; //->function
var type = typeof f2; //->function
var type = typeof f3; //->function
var type = typeof o1; //->object
var type = typeof o2; //->object
var type = typeof o3; //->object
```

---
## 2 构造函数

使用function定义的函数，可以当作普通函数使用，但是也可以通过该函数创建对象。此时该函数称为**构造函数**。

下面代码，person1 和 person2是通过构造函数Person构造的普通对象，person1 和 person2其实并没有constructor这个属性，这是属性是属于Person的原型对象的。只是person1 和 person2和可以继承原型对象的constructor这个属性。person1 与Person2这两个实例与构造函数Person并没有直接关系，person1 与Person2与Person的原型对象才有直接关系。

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
}

var person1 = new Person("mike", 12, "student");
var person2 = new Person("luck", 22, "teacher");
var same = (person1.constructor === person2.constructor && person2.constructor === Person);//->true，实例的构造函数属性（constructor）指向构造函数
var same = Person.prototype.constructor === Person;//->true

var object1 = new Object();
var object2 = {};
var same = object1.constructor === object2.constructor && object1.constructor === Object;//->true，普通对象的构造函数为Object
```

---
## 3 原型属性

在 JavaScript 中，每当定义一个对象（函数也是对象）时候，对象中都会包含一些预定义的属性。其中**每个函数对象都有一个prototype 属性**，这个属性指向函数的原型对象。**每个对象都有` __proto__`属性**，但只有函数对象才有 prototype 属性。

```javascript

function Person() {
}

Person.prototype.name = 'Jordon';
Person.prototype.age = 28;
Person.prototype.job = 'Software Engineer';

Person.prototype.sayName = function () {
    console.log(this.name);
};

var person1 = new Person();
person1.sayName(); //->Jordon

var person2 = new Person();
person2.sayName(); //->Jordon

var isSame = (person1.sayName === person2.sayName); //->true
```


----
## 4 什么是原型对象？

原型对象其实也是一个普通的对象。比如下面代码中的Person.prototype，Person.prototype其实还有一个默认的属性：constructor，在默认情况下，所有的原型对象都会自动获得一个 constructor（构造函数）属性，这个属性（是一个指针）指向 prototype 属性所在的函数，即：A 有一个默认的 constructor 属性，这个属性是一个指针，指向 Person。

```javascript
var A = Person.prototype;
console.log(Person.prototype.constructor === Person);//->true
console.log(person1.constructor === Person);//->true
console.log(Person.prototype === Person);//->false
```

注意：构造函数的原型对象只是构造函数的一个属性，这个属性是自动获得的。但原形对象并不是构造函数的一个实例。但是这个属性的构造函数指向该构造函数。

**原型对象其实就是普通对象**，但`Function.prototype` 除外，它是函数对象，但它很特殊，它没有prototype属性（前面说到函数对象都有prototype属性），但是`Function.prototype.__prote__ == Object.prototype`。

```javascript
function Worker() {
}

console.log(Worker.prototype); //->Worker{}，注意：Worker的prototype并不是Worker构造函数的一个实例
console.log(new Worker()); //->Worker{}
console.log(typeof Worker.prototype); //->Object
console.log(typeof Function.prototype); //-> Function，这个特殊
console.log(typeof Object.prototype); // ->Object
console.log(typeof Function.prototype.prototype); //->undefined
console.log(typeof Function.prototype.__proto__); //->object
console.log(Function.prototype.__proto__ === Object.prototype); //->true
```

---
## 5 `__proto__`属性

JS 在创建对象（不论是普通对象还是函数对象）的时候，都有一个叫做`__proto__` 的内置属性，用于指向创建它的构造函数的原型对象。

对象 person1 有一个 `__proto__`属性，创建它的构造函数是 Person，构造函数的原型对象是 Person.prototype ，所以：` person1.__proto__ === Person.prototype。`

关于`__proto__`属性：因为绝大部分浏览器都支持`__proto__`属性，所以它才被加入了 ES6 里（ES5 部分浏览器也支持，但还不是标准）。

```javascript
function Person() {
}

Person.prototype.name = 'Jordon';
Person.prototype.age = 28;
Person.prototype.job = 'Software Engineer';

Person.prototype.sayName = function () {
    console.log(this.name);
};

var person1 = new Person();


var isSame = (person1.__proto__ === Person.prototype);//->true
var isSame = Person.prototype.constructor === Person;//->true
var isSame = person1.__proto__ === Person.prototype;//->true
var isSame = person1.constructor === Person;//->true
```


------------
## 6 构造器

`var obj = {};` 与 `var obj = new Object();`是等效的，新对象obj是使用 `new` 操作符后跟一个构造函数来创建的。构造函数（Object）本身就是一个函数（即函数对象），**构造函数（Object）与其他构造器并没有什么区别**。只是它是js解释器中定义的。同理，可以创建对象的构造器不仅仅有 Object，也可以是 `Array，Date，Function`等。所以我们也可以通过构造函数来创建` Array、 Date、Function`实例。

```javascript
var b = new Array();
console.log(b.constructor === Array);//-> true
console.log(b.__proto__ === Array.prototype);//-> true

var c = new Date();
console.log(c.constructor === Date);//-> true
console.log(c.__proto__ === Date.prototype);//-> true

var d = new Function();
console.log(d.constructor === Function);//-> true
console.log(d.__proto__ === Function.prototype);//-> true

console.log(Function.constructor === Function);//->true，Function的constructor是其自身
console.log(Object.constructor === Function);//->true
console.log(Date.constructor === Function);//->true
console.log(RegExp.constructor === Function);//->true
console.log(Array.constructor === Function);//->true
console.log(function () {
    }.constructor === Function);//->true
```

---
## 7 函数对象

**所有函数对象的`__proto__`都指向Function.prototype**，它是一个空函数。所有函数对象的构造函数默认都指向Function。所有对象的 `__proto__` 都指向其构造器的 prototype。

**Function.prototype是唯一一个typeof 值为 function的prototype。**

`Function.prototype.__prote__ == Object.prototype。= true`这说明所有的构造器同时也都是一个普通 JS 对象，可以给构造器添加、删除属性等。同时它也继承了`Object.prototype`上的所有方法：`toString、valueOf、hasOwnProperty`等。

`Object.prototype.__proto__ === null` ，因为已经到最顶层了。


```javascript
Number.__proto__ === Function.prototype;  //->true
Number.constructor == Function; //true

Boolean.__proto__ === Function.prototype; //->true
Boolean.constructor == Function; //true

String.__proto__ === Function.prototype;  //->true
String.constructor == Function; //true

Array.__proto__ === Function.prototype;   //->true
Array.constructor == Function; //true

RegExp.__proto__ === Function.prototype;  //->true
RegExp.constructor == Function; //true

Error.__proto__ === Function.prototype;   //->true
Error.constructor === Function; //true

Date.__proto__ === Function.prototype    //->true
Date.constructor == Function //true

// 所有的构造器都来自于Function.prototype，甚至包括根构造器Object及Function自身
Object.constructor == Function; //->true
Object.__proto__ === Function.prototype;  //->true

Function.constructor == Function; //->true
Function.prototype.constructor == Function; //->true
Function.prototype.constructor.prototype == Function.prototype;//->true
Function.__proto__ === Function.prototype; //->true

Function.prototype.__proto__ === Object.prototype;//->true
Date.prototype.__proto__ === Object.prototype;//->true
Array.prototype.__proto__ === Object.prototype;//->true
RegExp.prototype.__proto__ === Object.prototype;//->true
var object1 = {};
object1.__proto__ === Object.prototype;//->true
```

JavaScript中有内置(build-in)构造器/对象共计12个（ES5中新加了JSON），这里列举了可访问的8个构造器。剩下如`Global`不能直接访问，`Arguments`仅在函数调用时由JS引擎创建，**Math，JSON是以对象形式存在的**，无需new。它们的`__proto__`都是Object.prototype。如下

```javascript
Math.__proto__ === Object.prototype;  //->true
Math.construrctor == Object; //->true

JSON.__proto__ === Object.prototype;  //->true
JSON.construrctor == Object; //true
```

**所有的构造器都来自于 Function.prototype，甚至包括根构造器Object及Function自身。**所有构造器都继承了Function.prototype的属性及方法。如length、call、apply、bind，所有的构造器也都是一个普通 JS 对象，可以给构造器添加/删除属性等。同时它也继承了Object.prototype上的所有方法：`toString、valueOf、hasOwnPropert`y等。

---
## 8 prototype

在ECMAScript 核心所定义的全部属性中，最耐人寻味的就要数 prototype 属性了。对于 ECMAScript 中的引用类型而言，prototype 是保存着它们所有实例方法的真正所在。换句话所说，诸如 toString()和 valueOf() 等方法实际上都保存在 prototype 名下，只不过是通过各自对象的实例访问罢了。

原型对象是用来做什么的呢？**主要作用是用于继承**，通过构造函数构造出的实例对象，自动继承它的构造函数的prototype上的属性和方法。

JS 内置了一些方法供我们使用，比如：

```javascript
 对象可以用 constructor/toString()/valueOf() 等方法;
 数组可以用 map()/filter()/reducer() 等方法；
 数字可用用 parseInt()/parseFloat()等方法；
```

当我们创建一个对象时：` var person = new Object()`， person 继承了Object 的原型对象Object.prototype上所有的方法：hasOwnProperty、toString等。

 当我们创建一个数组时：`var array = new Array()`，array继承了Array 的原型对象Array.prototype上所有的方法。

```
console.log(Object.getOwnPropertyNames(Object.prototype));//Object.prototype中所有的属性(包括不可枚举的属性)的名集合

console.log(Object.getOwnPropertyNames(Array.prototype));//获取Array.prototype中所有的属性(包括不可枚举的属性)的名集合

console.log(Object.getOwnPropertyNames(Function.prototype));//Function.prototype中所有的属性(包括不可枚举的属性)的名集合
```


---
## 9 修改对象的prototype

默认情况下：
```
 function Person(name) {
 this.name = name
 }
 var p = new Person('jack')
```

`p.__proto__与Person.prototype，p.constructor.prototype`都是恒等的，即都指向同一个对象。但是可以修改函数对象的prototype属性。

```
function Person(name) {
    this.name = name
}

var p1 = new Person('jack');
console.log(p1.__proto__ === Person.prototype); // true
console.log(p1.__proto__ === p1.constructor.prototype); // true
console.log(p1.constructor === Person); // true

// 重写原型
Person.prototype = {
    getName: function () {
    }
};

var p2 = new Person('jack');

console.log(p2.__proto__ === Person.prototype); // true
console.log(p2.__proto__ === p2.constructor.prototype); // false
console.log(p2.constructor === Person); // false
console.log(p2.constructor === Object); // true
```

给Person.prototype赋值的是一个对象直接量`{getName: function(){}}`，使用对象直接量方式定义的对象其构造器（constructor）指向的是根构造器Object，
`Object.prototype`是一个空对象`{}`，`{}`自然与`{getName: function(){}}`不等。**当修改函数对象的prototype后，使用该函数对象构造的实例的构造函数也变了。**

再来看一个列子：
```
function A() {

}
var a = new A();
console.log(a.constructor);//->[Function: A]
console.log(A.prototype.constructor);//->[Function: A]
A.prototype = Date.prototype;
console.log(A.prototype.constructor);//->[Function: Date]
var b = new A();
console.log(b.constructor);//->[Function: Date]
```

可以看出，当构造函数的原型被修改时，实例对象的构造函数属性将不再是原来的构造函数，而是与构造函数绑定的原型有关。



## 10 原型链

原型和原型链是JS实现继承的一种模型。**原型链的形成是真正是靠__proto__ 而非prototype**，参考下面代码：

```
var animal = function () {
};
var dog = function () {
};

animal.price = 2000;
dog.prototype = animal;

var tidy = new dog();

console.log(dog.price); //undefined，dog.prototype指向animal，animal.price=2000，但是dog无法继承自己的prototype的属性

console.log(tidy.price); // 2000，tidy.__proto__指向animal，它继承了animal.price=2000。
```

对象之间的继承关系构成了原型链，比如这里的原型链是：`child-->Child.prototype-->Object.prototype`

```
function Child() {
}

var child = new Child();
console.log(child.constructor === Child);//-> true
console.log(Child.constructor === Function);//-> true
console.log(child.__proto__ === Child.prototype);//-> true
console.log(Child.prototype.__proto__ === Object.prototype);//-> true
```

---
## 11 总结

- 每个对象都有 `__proto__` 属性，但只有函数对象才有 prototype 属性
- JS 在创建对象（不论是普通对象还是函数对象）的时候，都有一个叫做`__proto__` 的内置属性，用于指向创建它的构造函数的原型对象。即所有对象的 `__proto__` 都指向其构造器的 prototype。
- 所有函数对象的`__proto__`都指向Function.prototype，它是一个空函数。所有函数对象的构造函数默认都指向Function。
- 所有的构造器都来自于 Function.prototype，甚至包括根构造器Object及Function自身。 所有构造器都继承了Function.prototype的属性及方法。如length、call、apply、bind，
- 所有的构造器也都是一个普通 JS 对象，可以给构造器添加/删除属性等。同时它也继承了Object.prototype上的所有方法：toString、valueOf、hasOwnProperty等。

```
Object.__proto__ === Function.prototype // true，Object 是函数对象，是通过new Function()创建的，所以Object.__proto__指向Function.prototype。

Function.__proto__ === Function.prototype // true，Function 也是对象函数，也是通过new Function()创建，所以Function.__proto__指向Function.prototype。

 Function.prototype.__proto__ === Object.prototype //true，Function.prototype是个函数对象，理论上他的__proto__应该指向 Function.prototype，就是他自己，自己指向自己没有意义。
 JS一直强调万物皆对象，函数对象也是对象，让 Function.prototype.__proto__ 指向Object.prototype。Object.prototype.__proto__ === null，保证原型链能够正常结束。
```

---
## 12 null值

`ypeof(null)`的值是"object"，但是null 没有 `_proto_`。为什么？ null 是一个独立的数据类型，为什么typeof(null)的值是"object"？

- null不是一个空引用, 而是一个原始值， 它只是期望此处将引用一个对象, 注意是"期望", [null](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)
- typeof null结果是object, 这是个历史遗留bug。
- 在ECMA6中, 曾经有提案将type null的值纠正为null, 但最后提案被拒了，理由是历史遗留代码太多。


---
## 引用

- [最详尽的 JS 原型与原型链终极详解](http://www.jianshu.com/p/dee9f8b14771)
- [继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
