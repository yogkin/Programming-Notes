## 类型检测

判断对象类型的方法

- typeof：适用于函数对象和基本类型，遇到null失效
- instanceof：基于原型链，不同Window和iframe间是适用于instanceof
- `Object.prototype.toString`方法
    - `[object Array]`
    - `[object Function]`
    - `[object Null]`
    - `[object Undefined]`
    - `[object Object]`
    - IE6、7、8中，Null和Undefined的toString都返回`[object Object]`
- constructor
- duck type：鸭子辩型，不关心对象具体是什么类型，只关心对象是否具有某种功能。

```javascript
typeof window
"object"
Object.prototype.toString.call(window)
"[object Window]"
Object.prototype.toString.call(null)
"[object Null]"
Object.prototype.toString.call(undefined)
"[object Undefined]"
Object.prototype.toString.call({})
"[object Object]"
function Fa(){}
Object.prototype.toString.call(Fa)
"[object Function]"
```

## 运算符

- 一元运算符：++a、
- 二元运算符：a+b
- 三元运算符：b ? a:c
- `void(0)` 或 `void 0` 返回undefined

## 作用域

目前，JavaScript中没有块级作用域：

```javascript
//函数内
var a = b = 1;//b是全局变量
var a = 1, b=1;//b是函数变量
```

## for in

对对象和数组进行遍历时：`for in`不保证遍历的顺序

## 删除属性

某些属性不能被删除

```javascript
var globalVal = 1;
delete globalVal;//false

(function(){
    val localVal = 1;
    return delete localVal;
}());//flase

function fn(){}
delete fn;//false

ohNo = 1;
window.ohNo;//1
delete ohNo;//true
```

## 函数

- 函数声明会被前置：`function f(){}`，不能立即调用。
- 函数表达式不会前置`var f = function(){}`。

## ES 3执行上下文

- VO：每个作用域都有一个VariableObjec，简称VO，VO引用了所有作用域内的属性。
- AO，arguments object

VO的顺序填充：
1. 函数参数
2. 函数声明
3. 变量声明，初始化值为undefined

```javascript
function test(a,b){
    var c =10;
    function d(){}
    var e = function _e(){};
    (function x(){});
    b = 20;
}

调用：test(10)；

AO的填充顺序为：

AO(test) = {
    a:10，
    b:undefined，
    b:undefined，
    d:<ref to func "d">    
    e:undefined
}
```

## 引用

- [JavaScript深入浅出](https://www.imooc.com/learn/277)
