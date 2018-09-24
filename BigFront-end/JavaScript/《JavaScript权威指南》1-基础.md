
---
# 1 JavaScript概述

JavaScript是前端工程师必需掌握的三种技能之一，这三个技能分别为：

- 描述网页内容的HTML
- 描述网页样式的CSS
- 描述网页行为的JavaScript

JavaScript由解释器(引擎)解释运行，解释器一般由浏览器提供，比如Chrome的JavaScript解释器名称为V8，JavaScript的输入输出(类似网络、存储、图形相关的复杂特性)功能由**宿主环境**提供，**宿主环境**一般指浏览器。

JavaScript主要包括三个部分内容：

- JavaScript核心(ECMAScript)
- DOM 操作HTML文档
- BOM 操作浏览器

JavaScript核心内容包括

- 注释、字符集
- 变量、赋值、运算符、类型转换
- 函数：带有名称和参数的JavaScript代码段
- 对象、数组
- 正则表达式
- 面向对象编程

客户端JavaScript核心

- JavaScript如何在浏览器端运行
- Web浏览器端脚本技术，客户端JavaScript一些重要的全局函数
- 通过脚本来操作HTML内容
- 通过脚本来操作CSS
- 如何定义、注册事件处理程序

额外内容

- Jquery库
- 客户端存储机制
- Canvas


---
# 2 词法结构

- JavaScript采用Unicode字符集。
- JavaScript的保留字非常多，除此之外，ES3将Java的所有关键字都列为自己的保留字，虽然在ES5中有所放宽，但应该尽量不要使用这些关键字。
- 每一个JavaScript的运行环境(Client, Server)都有一个自己的全局属性列表，这也是需要注意的。
- JavaScript是分号可选的。但是应该在每个语句结束后加上分号，保证后续的代码压缩不会出现问题。
- return、break、continue后面不能换行。

示例：

```javascript
//return、break、continue后面不能换行
    return
    true
//将会被解析为
    return;ture;

//return、break、continue后面不能换行
x
++
y
//会被解析为
x; ++y
```

---
# 3 类型值和变量

JavaScript的变量类型分为**原值类型和对象类型**，原始类型包括：**数字，布尔值，字符串、null、undefined，除此之外都是对象类型**。

**对象是属性的集合**。每个属性都由名/值对组成。其中只有null和undefined是无法拥有方法的值。

---
## 3.1 数字

JavaScript不区分浮点数和整数。ES5严格模型中八进制是命令禁止的。JavaScript中内置的Math对象提供了许多常用数学运算的API。

JavaScript中的溢出、下溢或被零整除不会报错：

- `Infinity`表示正无穷大，`-Infinity`表示无负穷大。
- 下溢是指计算结果无穷接近于0，比JavaScript所能表达的最小的数还要小的值，这时JavaScript返回0，当负数发生下溢时返回-0。
- 被零整除时JavaScript不报错，只是返回Infinity或者-Infinity，
- `0/0`时没有意义的，返回一个NaN值。**NaN(not a number)**表示不是一个数字
- **NaN**是一个特殊的值，它和任何值都不相等，包括它自身，所以对于数字应该使用```if(x != x){...}```方式判断，因为当且仅当`X==NaN`时为true。
- `isNaN()`函数可以用来判断NaN，**这个方法并不可靠**
- 对于`isFinite()`函数，当参数不是NaN、Infinity、-Infinity时返回true。
- Number是数字的包装类型，在Number定义了一些数字常量，比如：`Number.NaN、Number.MAX_VALUE`。

```javascript
//定义数字
var num1 = 123;
var num2 = 0x123;
var num3 = 3.32;
var num4 = .324;
var num5 = 6.02e23;

//Math
console.log("3的9次方= " + Math.pow(3, 9));

//溢出、下溢
console.log(0 / 0);//->NaN
console.log(1 / 0);//->Infinity
console.log(-1 / 0);//-> -Infinity

//判断NaN的方法，JavaScript提供了isNaN来检测某个值是否为NaN，但是，这也不太精确的，
//因为，在调用isNaN函数之前，本身就存在了一个隐式转换的过程，它会把那些原本不是NaN的值转换成NaN的
var nan = NaN;
var same = nan !== nan;//->true
var same = isNaN(nan);//->true
isNaN("foo"); // true
isNaN(undefined); // true
isNaN({}); // true
isNaN({ valueOf: "foo" }); // true


//isFinite：不是NaN、Infinity、-Infinity时为true
isFinite(nan);//->false

//全局对象
console.log(Number.NaN);
console.log(Number.MAX_VALUE);
console.log(Number.MIN_VALUE);
console.log(Number.POSITIVE_INFINITY);
console.log(Number.NEGATIVE_INFINITY);
console.log(Math.PI);

//Number方法
var number = Number("23");
console.log(number);

//进制转换
var a = 15;
console.log(a.toString(2));//转化为二进制字符串，2表示转换为2进制
console.log(a.toString(8));
console.log(a.toString(16));
```


---
## 3.2 日期和时间

使用JavaScript的内置对Date来处理日期和时间

```
var date = new Date();//->2017-09-21T15:35:21.286Z
```

---
## 3.3 字符串

### 定义字符串(单引号与双引号)

字符串是原始类型，MCMAScript5中允许字符串直接量拆分成多行，每行必需以`\`结束

```
"
long\
line\
"
```

可以使用单引号和双引号定义字符串，单引号和双引号在各自单独用时是木有什么区别的，都可以，单引号和双引号混合使用时，这时候要特别注意了，这种情况一般出现在js拼接字符串里面，或者html元素的属性里面，就以JS为例吧，都是一样的规则：单引号和双引号必须成双成对的出现，可以单引号在外面，也可以双引号在外面：

```
var a="'你好'";//这里变量a的内容就是字符串'你好',这里的单引号也是字符串的一部分
var b='"你好"';//这里变量b的内容就是字符串"你好",这里的双引号也是字符串的一部分
console.info(a===b);//输出false，它们不是一样的字符串
```

### 字符串转换

其他数据类型可以直接使用`+`连接字符串。

```
var num = 123;
var numStr1 = ""+num;
var numStr2 = String(num);

num.toString(2);//数字num转为字符串形式，参数2表示使用2进制表示
```

### 字符串常用函数

```
//slice表示切开，可以反着切
console.log(str.slice(1, 4));
console.log(str.slice(-3));

//取子串
str.substring(1, 5);//参数：开始索引（含），结束索引（不含）
str.substr(1, 5);//参数：开始索引（含），取几个

//返回字符串第一个字符的ASCII码
str.charCodeAt(1);

//索引
console.log(str.charAt(3));//->l
console.log(str[3]);//->l

//正则表达式
var text = "testing: 1, 2, 3";
var pattern = "[s]";
console.log(text.search(pattern));//->2
```

---
## 3.4 布尔值

布尔值只有true和false，**JavaScript中的所有数值不是真值就是假值**，当JavaScript期望一个布尔值时，真值自动转换为true，而假值自动转换为false。

当布尔值需要转换为真假值时，true转换为1，false转换为0。

```javascript
var a = true;
var b = false;
var c = 234;

//空值判断
if (c !== null) {
    console.log("c != null");
}

//真值
if (c) {//->true
    console.log("c 是一个真值");
}

//转换
console.log(a + 4);//a转换为1，结果为5
console.log(b + 4);//a转换为0，结果为4
```

---
## 3.5 null和undefined

**null**是关键字，表示一个特殊值：空值。

**undefined**不是关键字，是JavaScript内置的一个全局对象，它的值就是未定位，它表示更深层次的null值，ES3中它的值是可读可写的，这个错误在ES5中被修复。

- undefined表示变量没有初始化
- 查询对象或者数组的原始返回undefined时，表示它们不存在
- 如果函数没有返回值，那么返回值就是undefined

unll和undefined都表示值是空缺的，`null==undefined`返回的true，使用恒等判断`===`才能区分它们。

可以认为undefined表示系统级的，出乎意料的或类似错误的值的空缺，**而null是程序级的，正常或者意料之中的值的空值**。如果想将它们赋值给变量，或者作为函数的参数，最佳选择是null。


```javascript
var valueNull = null;
var valueUndefined = undefined;
console.log(typeof valueNull);//->object
console.log(typeof valueUndefined);//->undefined
```

---
## 3.6 全局对象

全局对象的属性是全局定义的符号，在JavaScript中可以直接使用，JavaScript解释器启动时，就会创建好它们。

### 全局属性

- Infinity 代表正的无穷大的数值。
- NaN 指示某个值是不是数字值。
- undefined 指示未定义的值。

### 全局函数

- `decodeURI()`解码某个编码的 URI。
- `decodeURIComponent()`解码一个编码的 URI 组件。
- `encodeURI()`把字符串编码为 URI。
- `encodeURIComponent()`把字符串编码为 URI 组件。
- `escape()`对字符串进行编码。
- `eval()`计算 JavaScript 字符串，并把它作为脚本代码来执行。
- `getClass()`返回一个 JavaObject 的 JavaClass。
- `isFinite()`检查某个值是否为有穷大的数。
- `isNaN()`检查某个值是否是数字。
- `Number()`把对象的值转换为数字。
- `parseFloat()`解析一个字符串并返回一个浮点数。
- `parseInt()`解析一个字符串并返回一个整数。
- `String()`把对象的值转换为字符串。
- `unescape()`对由 escape() 编码的字符串进行解码。

示例：

```javascript
console.log(parseInt("3 abc")); //3
console.log(parseInt(" 3.14 abc")); //3
console.log(parseInt(" -3.14 abc")); //-3
console.log(parseInt("0xFF")); //255
console.log(parseFloat(".1")); //0.1
console.log(parseInt("0.1")); //0
console.log(parseInt(".1")); //NaN
console.log(parseFloat("$1")); //NaN
```


### 构造函数

比如`Date()、RegExp()、String()、Object()、Array()`，另外`Math和JSON`以对象的形式提供。

### 全局对象

>全局对象是预定义的对象，作为 JavaScript 的全局函数和全局属性的占位符。通过使用全局对象，可以访问所有其他所有预定义的对象、函数和属性。全局对象不是任何对象的属性，所以它没有名称。

>在顶层 JavaScript 代码中，可以用关键字 this 引用全局对象。但通常不必用这种方式引用全局对象，因为全局对象是作用域链的头，这意味着所有非限定性的变量和函数名都会作为该对象的属性来查询。例如，当JavaScript 代码引用 parseInt() 函数时，它引用的是全局对象的 parseInt 属性。全局对象是作用域链的头，还意味着在顶层 JavaScript 代码中声明的所有变量都将成为全局对象的属性。

>全局对象只是一个对象，而不是类。既没有构造函数，也无法实例化一个新的全局对象

>在 JavaScript 代码嵌入一个特殊环境中时，全局对象通常具有环境特定的属性。实际上，ECMAScript 标准没有规定全局对象的类型，JavaScript 的实现或嵌入的 JavaScript 都可以把任意类型的对象作为全局对象，只要该对象定义了这里列出的基本属性和函数。


在代码层次的最顶级，可以使用this引用全局对象，比如`var global = this`，在浏览器窗口JavaScript中，Window充当了全局对象。如果在代码中声明了一个全局属性，那么这个属性属于全局对象。

---
## 3.7 包装对象

JavaScript对象是一种复合值，它是属性或已命名值的集合。参照下面代码

```
var s = "abc";
var length = s.lenght;
```
我们知道字符串不是对象，但是为这么它具有属性呢？在JavaScript中只要引用了字符串`s`的属性，JavaScript就会将那个字符串值通过`new String()`的方式转换为对象，这个对象继承了字符串的方法，并被用来处理字符串的引用，一旦属性引用结束，这个对象也就会被销毁(实现可能不一致，但程序员看到的就是这样的)。

```
var s = "test";//创建一个字符串
s.len = 5;//给s设置一个属性
var t = s.len;
console.log(t);//打印结果为undefined
```

上面打印结果为undefined，因为第二步设置len属性时，创建了临时的字符串对象，随机字符串对象被销毁，所以第三步通过原始值s创建的另一个对象获取len属性，返回undefined。

---
## 3.8 类型转换

JavaScript隐式的类型转换与操作的数据类型和操作符都有关系：

值 | 转换为：字符串 | 转换为：数组 | 转换为：布尔值 | 转换为：对象
---|---|---|---|---
undefined<br/>null|"undefined"<br/>"null"|NaN<br/>0|false<br/>false<br/>|throws TypeError<br/>同上
true<br/>false|"true"<br/>"false"|1<br/>0| |new Boolean(true)<br/>new Boolean(fale)
""(空字符串)<br/>"1.2"<br/>"one"| |0<br/>1.2<br/>NaN|false<br/>true<br/>true|new String("")<br/>new String("1.2")<br/>new String("one")
0<br/>-0<br/>NaN<br/>Infinity<br/>-Infinity<br/>1|"0"<br/>"0"<br/>"NaN"<br/>"Infinity"<br/>"-Infinity"<br/>"1"|fales<br/>false<br/>false<br/>true<br/>true<br/>ture| |new Number(0)<br/>new Number(-0)<br/>new Number(NaN)<br/>new Number(Infinity)<br/>new Number(-Infinity)<br/>new Number(1)
{}：任意对象<br/>[]：任意数组<br/>[9]：一个数字的数组<br/>['a']：其他数组<br/>function(){}：任意函数|参考下面章节<br/>""<br/>"9"<br/>使用join()方法<br/>参考下面章节(函数也是对象)|参考下面章节<br/>0<br/>9<br/>NaN<br/>NaN|true<br/>true<br/>true<br/>ture<br/>true|



- `==`操作符会造成数据类型隐式的类型转换，对比较的结果会有影响，null和undefined例外，当它们用在一个期望是对象的地方将会导致TypeError异常。
- 虽然JavaScript可以自动做许多类型转换，但有时也需要程序然做显示的类型转换，常用到的方法有：`Number()、Boolean()、String()、Object()`。
- Number的toString方法可以计算一个基数，表示转换的进制。
- 对象转换为原始值，所有对象继承的两个方法为`toString和valueOf`，转换和这两个方法有关。

### 转换和相等性

由于JavaScript可以做灵活的类型转换，因此其"==”相等运算符也随相等的含义炅活多变，例如，如下这些比较结果均是true:

```javascript
null == undefined //这两值被认为相等
"0" == 0 //在比较之前字符单转换成教卞
o == false //在比tt之前布尔值转换成数卞
"0" == false //在比较之前字符串和布尔值都转換成数字
```

需要注意的是，一个值转换为另一个值并不意味着两个值相等，比如，如果在期望使用布尔值的地方使用了undefined，它将会转换为false，这并不代表`undefined == false`， JavaScript运算符和语句期望使用多样化的数据类型，并可以相互转换，if语句将`undefined`转换为false, 但是`==`运算符从不**试图**将从操作数转换为布尔值。

### 对象转换为原始值

1. 对象转为布尔值
1. 对象转换为字符串
 - 如果对象具有toString方法，则调用这个方法，如果该方法返回一个原始值，JS将这个值转为字符串，并返回这个值的结果
 - 如果对象没有toString方法或这个方法不返回一个原始值，那么Js调用其valueOf方法，如果返回值是原始值，JS将这个值转为字符串，并返回这个值的结果
 - 否则Js无法从toString或valueOf方法获得原值值将抛出类型错误
1. 对象转换为数字
 - 如果对象具有valueOf方法，该方法返回一个原始值，JS将这个值转为数字，并返回这个值的结果
 - 如果对象具有toString方法，该方法返回一个原始值，JS将这个值转为数字，并返回这个值的结果
 - 否则抛出类型错误

```javascript
//number to string
var num = 15.555;
console.log(num.toString(2));//->1111.10001110000101000111101011100001010001111010111
console.log(num.toFixed(2));//->15.55，保留小数点的位数换为字符串
console.log(num.toExponential(3));//->1.555e+1，使用指数计数法换为字符串
console.log(num.toPrecision(10));//->15.55500000，指定有效数字的位数转换为字符串

//========================================================================
// parseInt，parseFloat是全局函数
//========================================================================
console.log(parseInt("4 abc"));//4
console.log(parseInt("-12.32"));//-12
console.log(parseFloat("4.32 abc"));//4.32
console.log(parseFloat("0xFF"));//0

//========================================================================
// 对象转换为原始类型
//========================================================================
//对象转换为原始类型用到的方法是toString和valueOf
console.log([1, 3, 4].toString());//1,3,4
console.log(new Date().valueOf());//1525771638646
```

---
## 3.9 变量声明

JavaScript使用**var**来声明变量。如果声明一个变量但是不初始化，那么它的值就是undefined。如果声明变量时不使用var，那么这个变量就是全局变量。在函数内部也可以省略var来声明全局变量。

JavaScript是弱类型动态语言。变量可以任意类型。`var i = 10;i = "ten";`是完全合法的。使用var重复声明变量也是合法无害的。

---
## 3.10 变量作用域、函数作用域和声明提前

- JavaScript根据变量作用域分为**全局变量和局部变量**
- 函数的定义是可以嵌套的，每个函数都有自己的作用域
- JavaScript没有类似C、Java的块级作用域，取而代之的是**函数作用域**
- 函数作用域的现象就是声明提前

声明提前实例：
```
var a = "abc"//全局变量
function test(o) {
    console.log(a);//输出undefined，函数内变量声明提前
    var i = 0;
    if (typeof 0 === 'object') {
        var j = 0;
        for (var k = 0; k < 10; k++) {
            console.log(k);
        }
        console.log(k);
        var a = "bcd";
    }
    console.log(j);//合法，j已经定义了，只是没有初始化，输出undefined。
}
```

---
## 3.11 作用域链

JavaScript是基于**词法作用域**的语言，通过阅读包含变量定义在内的数行代码就能知道变量的作用域，每一段JavaScript代码(全局代码或者函数)都有一个与之关联的作用域链，作用域链是一个对象列表或者链表。变量解析是基于作用域链查找的。

---
# 4 表达式

## 4.1 创建对象

- `{}`和`[]`初始化表达式分别用于创建对象和数组
- 函数表达式：用于创建函数
- 属性访问表达式：包括 `[] `和` .`
- 对象创建表达式：使用`new`创建对象


```javascript
//========================================================================
// 对象和数组初始化表达式
//========================================================================
var emptyArray = [];
var matrix = [[1, 2, 3], [1, 2, 3], [1, 2, 3]];//模拟二维数组
var sparseArray = [1, , , , undefined, 3];//中间使用undefined填充


//定义对象，本质上等效于new Object();
var objP = {
    x: 2.3, y: -1.2
};

//函数表达式
var square = function (x) {
    return x * 2;
};

//========================================================================
// 属性访问表达式，包括 [] 和 .
//========================================================================
var objQ = {x: 1, y: 2, z: {a: 3}};
console.log(objQ.x);//->1
console.log(objQ.z.a);//->3
console.log(objQ["x"]);//->3
```

---
## 4.2 运算符表达式

操作数和结果类型：一些运算符可以作用于任何类型，但仍希望它们操作的数是指定类型的，并且大多数操作符返回特定类型的值

### `+ `运算符

如果有一个数据类型是字符串，那么返回字符串，如果两个操作数都是对象，那么都将进行数学运算。

```javascript
//操作数和结果类型：一些运算符可以作用于任何类型，但仍希望它们操作的数是指定类型的，并且大多数操作符返回特定类型的值

//比如 * 希望操作数字类型，下面表达式的结果是数字9
var result = "3" * "3";//->9，type为number

//+ 运算符：如果有一个数据类型是字符串，那么返回字符串，如果两个操作数都是对象，那么都将进行数学运算。
var date = new Date();
var a = 3;
console.log(a + {});//->3[object Object]，返回的结果是字符串，{}对象转换为字符串了
console.log(date + a);//->Fri Sep 22 2017 00:10:03 GMT+0800 (中国标准时间)3，结果为string
console.log(2 + null);//->2，null->0
console.log(2 + undefined);//->NaN
```

### `==与===`关系运算符

两个操作符都可以比较任意类型，`===`更为严格相等运算符(也叫恒等于)，`==`是相等运算符

- `===`在比较时不做类型转换，与之对于的不等比较符号是 `!==`
- `==`在比较时会做类型转换，与之对于的不等比较符号是 `!=`
- JavaScript对象的比较是引用比较，而不是值比较，对象只和本身相等，和其他任何对象都不相等

```javascript
var objA = {};
var objB = {};
console.log(objA == objB);//false

//String在进行比较时，应该选择String.localCompare()函数
console.log("abc".localeCompare("abc"));//0
```

### 比较运算符(`<、>、>=、<=`)

只有数字和字符串才能进行真正意义上的比较，其他类型都将进行类型转换。如果至少有一个操作数不是字符串，那么两个操作数都转换为数字进行比较，可以说`+`运算符更加偏爱字符串，但是比较运算符更加偏爱数字

```javascript
console.log("11" > 3);//数字的比较
console.log("one" < 0);//数字的比较，one->NaN，始终是false
```

### in运算符

in运算符希望左操作数是一个字符串或者可以转换为字符串，希望它的右操作数是一个对象，如果对象中有一个名为左操作数的属性名，返回true。

```javascript
console.log("x" in {x: 32, y: 32});
```

### instanceof运算符

instanceof运算符希望左操作数是一个对象，右操作数是定义类的构造函数，为了了解instanceof是如何工作的，必需了解原型链(prototype chain)

```javascript
//instanceof运算符希望左操作数是一个对象，右操作数是一个类，为了了解instanceof是如何工作的，必需了解原型链(prototype chain)，原型链是JavaScript的继承机制，
//为了计算o instanceof f，JavaScript首先计算f.prototype, 然后在原型链中查找o，如果找到那么o是f(或f的父类)的一个实例。返回true，否则返回false

var d = new Date();
console.log(d instanceof Date);
console.log(d instanceof Object);//Object是所有对象的父类
console.log(d instanceof Number);//false
```

### 逻辑表达式`&& 、|| 、！`

```javascript
//逻辑表达式操作的数据类型是boolean是很好判断的，单当操作的数据类型不是boolean类型时，就会根据数据是真假值进行判断
var objC = {x: 3};
var empty = null;
console.log(objC && empty);
```

### eval函数

JavaScript可以解释运行由JavaScript源代码组成的字符串，并产生一个值，这个功能由全局函数`eval()`来完成

```javascript
//JavaScript可以解释运行由JavaScript源代码组成的字符串，并产生一个值，这个功能由全局函数eval()来完成
console.log(eval("3+32"));  // 35
```

### typeof操作符

返回表示**操作数类型**的一个字符串

```javascript
//typeof操作符：返回表示操作数类型的一个字符串
//注意：虽然函数是对象，但是JavaScript对其做了特殊对待，typeof 一个函数返回的是 ”function“

function func() {

}
console.log(typeof func);//->function
```

### delete

delete操作符用于删除对象属性或者数组元素

```javascript
var o = {x: 1, y: 2};
delete o.x;
console.log("x" in o);//false
```

### void

void是一个运算符，出现在操作数之前，操作数可以是任何类型，操作符会照常计算，但忽略计算结果并返回undefined，由于void会忽略操作数的值，因此在操作数具有副作用的时候用void来让程序更有意义，比如：`<a href="javascript:void window.open();">打开窗口</a>`


### 关于操作符隐含的类型转换

涉及隐式转换最多的两个运算符 `+` 和 `==`，**这些运算符只会针对number类型，故转换的结果只能是转换成number类型。**

---
# 5 语句

---
## switch

switch语句的条件判断使用的是`===`

```javascript
function testSwitch(x) {
    switch (x) {
        case 1:
            console.log("不满意");
            break;
        case 2:
            console.log("满意");
            break;
        default:
            console.log("不是标准答案");
            break;
    }
}
```

---
## with

with语句用于临时扩展作用域链，**但不建议使用**

```javascript
var values = {a: 1, b: 2, c: 3};
with (values) {
    console.log(a);
    console.log(b);
    console.log(c);
}
```

---
## fot in

`for in`语句用于遍历可迭代的对象

```javascript
var obj = {a: 1, b: 2, c: 3};
var value;
for (value in  obj) {
    console.log(value);
}

var arr = [], i = 0;
for (arr[i++] in obj);//用于复制obj中的数据到arr
console.log(arr);
```

---
## 异常处理语句

```javascript
try {
    var a = undefined;
    a.toString();
} catch (e) {
    console.log(typeof e);
    console.log(e);
} finally {
    console.log("error");
}

//throw可以抛出一个异常
throw "抛出字符串异常";
throw 1;
throw new Error("发生错误了");
```

---
## 其他

- `"use strict"`用于启动严格模式，在ES5中引入
- debugger语句用于添加断点