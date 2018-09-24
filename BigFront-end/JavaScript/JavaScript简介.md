# JavaScript简介

---
## 1 JavaScript的用途

JavaScript 最初用来制作 web 页面交互效果，提升用户体验。比如实现下面列出的效果：

- 轮播图
- Tab栏（选项卡）
- 地图
- 表单验证

## 2 JavaScript 历史背景介绍

布兰登-艾奇（Brendan Eich，1961年～），1995年在网景公司，发明的JavaScript。一开始JavaScript叫做LiveScript，但是由于当时Java这个语言特别火，所以为了傍大牌，就改名为JavaScript。如同 “北大” 和 “北大青鸟” 的关系。“北大青鸟” 就是傍 “北大” 大牌。同时期还有其他的网页语言，比如VBScript、JScript等等，但是后来都被JavaScript打败，所以现在的浏览器中，只运行一种脚本语言就是JavaScript。

## 3 JavaScript 和 ECMAScript的关系

ECMAScript是一种由Ecma国际前身为欧洲计算机制造商协会,英文名称是European Computer Manufacturers Association，制定的标准。

JavaScript是由公司开发而成的，公司开发而成的一定是有一些问题，不便于其他的公司拓展和使用。所以欧洲的这个ECMA的组织，牵头制定JavaScript的标准，取名为ECMAScript。简单来说ECMAScript不是一门语言，而是一个标准。符合这个标准的比较常见的有：JavaScript、Action Script（Flash中用的语言）。

- 2009年12月，ECMAScript 5.0版正式发布。
- ECMAScript在2015年6月，发布了ECMAScript 6(即ECMAScript 2015)版本，语言的能力更强。但是浏览器的厂商不能那么快的去追上这个标准。

## 4 今天的JavaScript：承担更多责任

- 2003年之前，JavaScript被认为“牛皮鲜”，用来制作页面上的广告，弹窗、漂浮的广告。什么东西让人烦，什么东西就是JavaScript开发的。所以浏览器就推出了屏蔽广告功能。
- 2004年JavaScript命运开始改变了，那一年谷歌公司，开始带头使用Ajax技术了，Ajax技术就是JavaScript的一个应用。并且，那时候人们逐渐开始提升用户体验了。
- 2007年乔布斯发布了iPhone，这一年开始，用户就多了上网的途径，就是用移动设备上网。JavaScript在移动页面中，也是不可或缺的。并且这一年，互联网开始标准化，按照W3C规则三层分离，人们越来越重视JavaScript了。
- 2010年的时候，人们更加了解HTML5技术了，HTML5推出了一个东西叫做Canvas（画布），工程师可以在Canvas上进行游戏制作，利用的就是JavaScript。
- 2011年，Node.js诞生，使JavaScript能够开发服务器程序了。

## 5 JavaScript主要内容

JavaScript分为几个部分：

- **语言核心(ECMAScript)** - 基础班只学习语言核心，变量、表达式、运算符、函数、if语句、for语句
- **DOM** - 控制HTML中的元素，比如让盒子移动、变色、轮播图。DOM是啥。
- **BOM** - 控制浏览器的一些东西，比如让浏览器自动滚动。
