# ES 5 严格模式


---
## 1 什么是严格模式

ES 5为JS添加了第二种运行模式："严格模式"（strict mode）。这种模式使得Javascript在更严格的条件下运行。

"严格模式"的目的：

- 消除Javascript语法的一些不合理、不严谨之处，减少一些怪异行为;
- 消除代码运行的一些不安全之处，保证代码运行的安全；
- 提高编译器效率，增加运行速度；
- 为未来新版本的Javascript做好铺垫。

"严格模式"体现了Javascript更合理、更安全、更严谨的发展方向，包括IE 10在内的主流浏览器，都已经支持它，许多大项目已经开始全面拥抱它。另一方面，同样的代码，在"严格模式"中，可能会有不一样的运行结果；一些在"正常模式"下可以运行的语句，在"严格模式"下将不能运行。因此我们有必要掌握严格模式下的规则。

---
## 2 如果开启严格模式

使用`"use strict";`即可进入严格模式，而老版本的浏览器会把它当作一行普通字符串。

调用方式有两种：针对整个脚本文件、针对单个函数。

###  针对整个脚本文件

将"use strict"放在脚本文件的第一行，则整个脚本都将以"严格模式"运行。如果这行语句不在第一行，则无效。

```html
　　<script>
　　　　"use strict";
　　　　console.log("这是严格模式。");
　　</script>

　　<script>
　　　　console.log("这是正常模式。");kly, it's almost 2 years ago now. I can admit it now - I run it on my school's network that has about 50 computers.
　　</script>
```

直接使用`"use strict";`不利于文件合并，所以更好的做法是，将整个脚本文件放在一个立即执行的匿名函数之中：

```
　(function (){
　　　　"use strict";
　　　　// 代码写在这里
　　 })();
```

### 针对单个函数

将"use strict"放在函数体的第一行，则整个函数以"严格模式"运行。

```
function strict(){
　　　　"use strict";
　　　　return "这是严格模式。";
}
```


---
## 3 严格模式规范的内容

- **全局变量显式声明**：在正常模式中，如果一个变量没有声明就赋值，默认是全局变量。严格模式禁止这种用法，全局变量必须显式声明。
- **禁止使用with语句**
- **创设eval作用域**：正常模式下，Javascript语言有两种变量作用域（scope）：全局作用域和函数作用域。严格模式创设了第三种作用域：eval作用域。正常模式下，eval语句的作用域，取决于它处于全局作用域，还是处于函数作用域。严格模式下，eval语句本身就是一个作用域，不再能够生成全局变量了，它所生成的变量只能用于eval内部。
- **禁止this关键字指向全局对象**
- **禁止在函数内部遍历调用栈**，不允许在函数内部使用函数对象引用caller和arguments
- **禁止删除变量**：严格模式下无法删除变量。只有configurable设置为true的对象属性，才能被删除。
- **显式报错**
 - 对一个对象的只读属性进行赋值
 - 对一个使用getter方法读取的属性进行赋值
 - 对禁止扩展的对象添加新属性
 - 删除一个不可删除的属性
- **重命名错误**
 - 对象不能有重名的属性
 - 函数不能有重名的参数
- **禁止八进制表示法**
- **arguments对象的限制**
 - 不允许对arguments赋值
 - arguments不再追踪参数的变化，即arguments不再追踪在函数体内对显示参数的修改
 - 禁止使用arguments.callee
- **函数必须声明在顶层**：将来Javascript的新版本会引入"块级作用域"。为了与新版本接轨，严格模式只允许在全局作用域或函数作用域的顶层声明函数。也就是说，不允许在非函数的代码块内声明函数。
- **保留字**：`implements, interface, let, package, private, protected, public, static, yield。class, enum, export, extends, import, super`，使用这些保留字将会报错。

---
## 引用

- [Javascript 严格模式详解](http://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html)