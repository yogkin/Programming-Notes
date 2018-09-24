# 7 数组

---
## 7.1 数组介绍

数组是值的有序集合，**js数组是js对象的特殊形式**。数组最大能容纳`2^32-1`个元素。js数组不支持多维数组，但是可以用数组的数组来模拟。

```javascrpt
var empty = [];
var primes = [2, 3, 5, 7, 11];
var misc = [1.1, true, "a"];
var byte = 1024;
var table = [byte, byte + 1];
var undefs = [, 1, , 3];//稀疏数组，不连续的数组
var arr = new Array(10);//长度为10
var arr1 = new Array(5, 4, 3, 2, 1, true);//指定元素
```

---
## 7.2 属性

ES5中数组可以定义setter和getter方法，此时访问这种数组的元素的时间会与普通对象属性的查找时间相近，也可以给数组对象定义普通属性，比如：创建一个名为`-1.21`的属性`a[-1.21] = true;`，本质上-1.21转换为字符串，它只是一个普通的属性。


```javascrpt
//属性的一些操作

var a = [];
a[-1.21] = true;//创建一个名为-1.21的属性，本质上-1.21转换为字符串，它只是一个普通的属性
a["10"] = 10;//给第10个元素赋值
a[1.00] = 3;//等价于a[1]
a[-2] = "-2";//添加一个名为-2的属性
console.log(a);//-> [ , 3, , , , , , , , , 10, '-1.21': true, '-2': '-2' ]
console.log(a[1]);
a.length = 0;//删除所有的元素
```

ES5中让数组的length变为不可写，此时无法操作数组

```javascrpt
Object.defineProperty(a, "length", {
    writable: false
});
console.log(a.length);
// a.push(2); //报错，Cannot assign to read only property 'length' of object '[object Array]'

var b = [3];
Object.freeze(b);
b.length = 0;//无效
console.log(b);
// b.push(2); //报错， Can't add property 1, object is not extensible
```

---
## 7.3 添加与访问数组元素的方法

- `push()` 尾部插入一个或多个元素
- `pop()` 尾部弹出一个元素
- `shift()` 头部删除一个元素
- `unshift()` 头部插入一个元素
- `splice()` 通用方法

```javascript
var c = [];
c.push(1);
c.push(1, 2, 4);
c.unshift("S");//在数组的头部加入一个元素

c[1] = "A";
var success = delete c[1];//仅仅是删除元素
var result = c.pop();//弹出元素，使长度减1，
var top = c.unshift();//头部弹出元素
```


---
## 7.4 遍历数组

 - for循环
 - for/in **不推荐，顺序可能没有被保证**。
 - forEach


```javascript
//使用for循环是遍历数组最常见的方法
var o = {x: 1, y: 2, z: 3};
var keys = Object.keys(o);
for (var i = 0, len = keys.length; i < len; i++) {
    if (!keys[i]) {//跳过null和undefined
        continue;
    }
    var value = keys[i];
    console.log(value);
}

//使用for/in循环能够枚举继承属性名，ES规定for/in循环可以以不同的顺序遍历对象的属性，通常数组的遍历是升序的，
//但是不能保证这是一定的，很多时候应该避免使用for/in来遍历数组
for (var element in keys) {
    if (!keys.hasOwnProperty(element)) {//跳过继承的属性
        continue;
    }
    console.log(element);
}

//使用ES5提供的forEach是一个不错的选择
keys.forEach(function (p1, p2, p3) {//分别是属性名、值、数组本身
    console.log(p1 + "  " + p2 + "  " + p3.toString());
});
```

## 7.5 常用函数

-  `concat()` 连接两个或更多的数组，并返回新的数组
-  `join()` 把数组的所有元素放入一个字符串。元素通过指定的分隔符进行分隔
-  `pop()` 删除并返回数组的最后一个元素
-  `push()` 向数组的末尾添加一个或更多元素，并返回新的长度
-  `reverse()` 颠倒数组中元素的顺序。会修改数组
-  `shift()` 删除并返回数组的第一个元素
-  `slice()` 从某个已有的数组返回选定的元素，返回新的数组
-  `sort()` 对数组的元素进行排序，会修改数组，还可以传入function对排序进行设定，负前正后
-  `splice()` 删除元素，并向数组添加新元素，splice是在数组中插入或者删除元素的通用方法，会修改数组，第一个参数表示插入或删除的起始位置，第二个参数表示从数组中应该删除的个数
-  `toSource()` 返回该对象的源代码
-  `toString()` 把数组转换为字符串，并返回结果
-  `toLocaleString()` 把数组转换为本地数组，并返回结果
-  `unshift()` 向数组的开头添加一个或更多元素，并返回新的长度
-  `valueOf()` 返回数组对象的原始值

```javascript
//join
var a = [1, 2, 3, 4, 5, 6];
console.log(a.join('--'));//->1--2--3--4--5--6

//反转数组，会修改数组
a.reverse();
console.log(a);//->[ 6, 5, 4, 3, 2, 1 ]

//sort
var bsort = ["z", "y", "x"];
b.sort();
console.log(b);//->[ 'x', 'y', 'z' ]

// concat连接两个或更多的数组，并返回新的数组
console.log(a.concat(3));//[ 6, 5, 4, 3, 2, 1, 3 ]
console.log(a.concat([3, 5], 9, [4, [4]]));//->[ 6, 5, 4, 3, 2, 1, 3, 5, 9, 4, [ 4 ] ]

// slice从某个已有的数组返回选定的元素，返回新的数组
console.log(a.slice(0, 3));//->[ 6, 5, 4 ]
console.log(a.slice(1, -1));//-1相对于最后一个数组，[ 5, 4, 3, 2 ]

//splice是在数组中插入或者删除元素的通用方法，会修改数组，第一个参数表示插入或删除的起始位置，第二个参数表示从数组中应该删除的个数
var a = [1, 2, 3, 4, 5, 6, 7, 8, 9];
var newA = a.splice(4);
console.log(newA);//->[ 5, 6, 7, 8, 9 ]
console.log(a);//->[ 1, 2, 3, 4 ]

var a = [1, 2, 3, 4, 5];
var newA = a.splice(2, 0, "a", "b");
console.log(a);//->[ 1, 2, 'a', 'b', 3, 4, 5 ]
console.log(newA);//->[]
console.log(a);//[ 1, 2, 'a', 'b', 3, 4, 5 ]

var newA = a.splice(2, 2, [1, 2], "b");
console.log(newA);//[ 'a', 'b' ]
console.log(a);//[ 1, 2, [ 1, 2 ], 'b', 3, 4, 5 ]
```

---
## 7.6 函数式

ES5中定义了9个新的数组方法来**遍历、映射、过滤、简化、搜索**数组。

  - `forEach`：接收一个function，用于遍历数组，不用使用break跳出，跳出只能制造异常
  - `map`：对数组的每一个元素进行变换，返回新的数组
  - `filter`：过滤数组中的元素，返回新的数组，filer会跳过稀疏数组中缺少的元素，总是返回稠密的数组
  - `every和same`：every和same返回true或者false，用于逻辑判断，some只要遇到条件为true的元素就会立即返回。
  - `reduce和reduceRight`：使用指定的方法对元素的数组进行组合，返回组合后的值，第一个参数为函数，第二个可选参数为传递给函数的初始化值
  - `indexOf和lastIndexOf`：索引元素

---
## 7.7  数组类型

Array.isArray()方法判断是否为数组。

```javascript
//ES5中用于判断数组的方法
var a = [1, 2, 3];
Array.isArray(a);//-> true

//ES3中却没有对应的方法，typeof只是简单的返回对象，instanceof操作只能用于简单的判断
var isInstanceofArray1 = [] instanceof Array;//->true
var isInstanceofArray2 = {} instanceof Array;//->false

//兼容ES3方式为
var isArrayFunc = Array.isArray || function (o) {
        return typeof o === "object" &&
            Object.prototype.toString.call(o) === "[Object Array]";
    };
console.log(isArrayFunc(a));//->true
console.log(Object.prototype.toString.call(a));//->[object Array]
```

---
## 7.8 类数组对象

js数组的特性：

 - 自动更新length
 - 设置length为较小的值时自动截断数组
 - 从Array.prototype继承一些方法
 - 类属型为“Array”

但这些都不是定义数组的本质特性，一种常常完全合理的看法是把有**一个length属性**和对应**非负整数属性**的对象看作一种类型的数组。即**类数组**。**js数组的方法是特意定义为通用的，因此它们可以用在类数组对象上**

```javascript
var a = {};
var i = 0;
while (i < 10) {
    a[i] = i * i;
    i++;
}
a.length = i;
//现在a就像一个数组，可以像数组一样遍历它
var total = 0;
for (var j = 0, length = a.length; j < length; j++) {
    total += a[j];
}

console.log(total);//->285
```

类数组对象：

- 字符串
- arguments

function的隐式`arguments`参数是一个类数组，DOM中`getElementsByTagName()`返回的是一个类数组

```javascript
//用于判断类数组的方法
function isArrayLike(o) {
    return !!(o && //是真值
    typeof  o === "object" &&//是对象
    isFinite(o.length) &&//有length属性
    o.length >= 0 &&//length大于等于0
    o.length === Math.floor(o.length) &&//是整数
    o.length < Math.pow(2, 32));//length < 2^32
}
```

js数组的方法是特意定义为**通用的**，因此它们可以用在类数组对象上，ES5中所有的数组方法都是通用的，ES3中除toString和toLocalString方法都是通用的。使用方式为`Function.call`

```javascript
var b = {"0": "a", "1": "b", "2": "c", length: 3};//类数组对象
var result = Array.prototype.join.call(b, "--");
console.log(result);//->a--b--c

//在FireFox这些方法的版本被定义在Array构造函数上，所以可以直接使用，但是这不是标准的，所以应该使用兼容的方式，比如：
Array.join = Array.join || function (a, sep) {
        return Array.prototype.join.call(a, sep);
    };
console.log(Array.join(b, "-"));
```

作为数组的字符串：

```
//字符串类似于只读数组，除了用charAt方法还可用[]直接访问字符，所以数组方法也可以用于字符串，除了push等会改变数组的方法除外，因为字符串是不可变对象。
var s = "JavaScript";
console.log(s[1]);
console.log(Array.prototype.join.call(s, '-'));
```
