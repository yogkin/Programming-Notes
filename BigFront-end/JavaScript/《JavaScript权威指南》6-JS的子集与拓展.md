
---
# 11 JavaScript的子集和拓展

>到目前为止，本书参照ECMAScript 3和ECMAScript 5中的标准规范完整地讨论了JavaScript这门官方语官，从现在起，本章将开始讨论JavaScript的子集和超集，其中子集的定义大部分都是出于安全考虑，只有使用这门语言的一个安全的子集编写脚本，才能让代码执行得更安全、更稳定，比如如何更安全地执行一段由不可信第三方提供的广 
告代码。

>ECMAScript 3标准是1999年颁布的，十年后，也就是2009年才更新到了ECMAScript 5，JavaScript的作者Brendan Eich在这十年间不断地改进这门语言（ECMAScript标准规范是允许对其做任何扩充的），同时，伴随若MoziHa项目的推进，在Firefox 1.0、1.5、2、3和3.5版本中分别发布了JavaScript 1.5、1.6、1.7、1.8和1.8.1版本.这些JavaScript的扩展版本中的很多新特性已经融入到ECMAScript 5中，还有很多特性依然是非标准的，但这些特性将有很大一部分会融入到ECMAScript的将来版本中。 由于Firefox是基于一个名叫Spidermonkey的JavaScript引擎因此Firefox浏览器也可以支持这些扩展特性。由MoziUa开发的另一个基于Java的JavaScript引擎Rhino也支持大部分扩展特性。但由于这些语言特性是非标准的，本章内容对于那些需要调试浏览器兼容性的开发者来说可能帮助不大。我们在本章对它们作必要的讲述是基于几点考虑：

- 它们足够强大
- 它们为了可能称为标准

子集和拓展内容如下：

- JavaScript子集
- 常量和局部变量
- 解构赋值
- 迭代
- 函数简写
- 多catch从句
- ES for xml

---
#  12 服务都 JS

## Rhino

Rhino是一种使用Java编写的JavaScript解释器。

- [rhino](https://github.com/mozilla/rhino)
- [tutorial](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino/Embedding_tutorial)

## NodeJS

- [nodejs](https://nodejs.org/zh-cn/)


