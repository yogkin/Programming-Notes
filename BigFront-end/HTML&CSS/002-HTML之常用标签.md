# HTML之常用标签

---
## 1 基本语法

- HTML对换行不敏感，对tab不敏感
- **空白折叠现象**：HTML中所有的文字之间，如果有空格、换行、tab都将被折叠为一个空格显示。
- 标签要严格封闭

>HTML、CSS就是写代码，不能算“编程”，因为这里面没有业务逻辑，加减乘除，与或非。说白了，就是用代码画画。

---
## 2 常用标签

具体参考[W3 HTML 参考手册](http://www.w3school.com.cn/tags/index.asp)

### 头标签

头标签都放在`<head></head>`头部分之间。头标签中的内容不会展示在网页中，头标签包括：

- `<title>`：指定浏览器的标题。
- `<base>`：为页面上的所有链接规标题栏显示的内容定默认地址或默认目标。
- `<meta>`：可提供有关页面的基本信息
- `<link>`：定义文档与外部资源的关系。

### h系列

`h`是容器级的标签。理论上里面可以放置`p、ul`，只是允许，在语义上，不要这么写。

### p标签

段落，是英语paragraph“段落”缩写。HTML标签是分等级的.

HTML将所有的标签分为两种：**容器级、文本级。**

- 容器级的标签，里面可以放置任何东西；
- 文本级的标签里面，只能放置**文字、图片、表单元素**。

**p标签是一个文本级标签**。记住：p里面只能放文字、图片、表单元素。其他的一律不能放。

### 图片

- 图片标签为`img`,img即image,img标签式自封闭的，仅代表图片
- alt属性：alt是英语`alternate`“替代”的意思，就表示不管因为什么原因，当这个图片无法被显示的时候，出现的替代文字（有的浏览器不支持）。

### 超级链接

- a是英语anchor“锚”的意思
- href是英语hypertext reference超文本地址的缩写
- 页面内的锚点`<a name="wdzp">我的作品</a>`，`<a href="#wdzp">`
- a是一个文本级的标签

### 列表

无序列表

 - ul就是英语unordered list，“无序列表”的意思
 - li 就是英语list item ， “列表项”的意思。
  - 所有的li不能单独存在，必须包裹在ul里面；反过来说，ul的“儿子”不能是别的东西，只能有li。
 - ul的作用，并不是给文字增加小圆点的，而是增加无序列表的“语义”的
 - li是一个容器级标签，li里面什么都能放

有序列表

- ordered list  有序列表，用ol表示

定义列表

- dl表示definition list 定义列表
- dt表示definition title    定义标题
- dd表示definition description 定义表述词儿
- dt、dd只能在dl里面；dl里面只能有dt、dd
- dt、dd都是容器级标签，想放什么都可以。

### div和span

div和span是非常重要的标签，div的语义是division“分割”； span的语义就是span“范围、跨度”，这两个东西，都是最重要的**“盒子”**

- div在浏览器中，默认是不会增加任何的效果改变的，但是语义变了，div中的所有元素是一个小区域
- div标签是一个容器级标签，里面什么都能放，甚至可以放div自己
- span也是表达“小区域、小跨度”的标签，但是是一个“文本级”的标签。
- span里面只能放置文字、图片、表单元素。 span里面不能放`p、h、ul、dl、ol、div。`
- span里面是放置小元素的，div里面放置大东西的

### 表单

表单标签：`<form>`用于与服务器端的交互表单标签：`<form>`用于与服务器端的交互，`<input>`：输入标签 ,接收用户输入`<input>`：输入标签 ，接收用户输入。

input包括以下类型：

- 文本框：`text`
- 密码框：`password`
- 单选按钮：`radio`
- 复选框：`checkbox`
- 下拉列表：`select`
- 多行文本框：`textarea`
- 三种按钮
 - button
 - submit
 - reset
 - label

### 表格

- table标签常用属性：`border、width、height、align、bgcolor、cellpadding、cellspacing`
- caption用于设置标题
- 一般情况下thead、 tbody、 tfoot不会被用到
- thead、 元素应该与 tbody 和 tfoot 元素结合起来使用。
- tbody 元素用于对 HTML 表格中的主体内容进行分组，而 tfoot 元素用于对 HTML 表格中的表注（页脚）内容进行分组。
- 如使用 thead、tfoot 以及 tbody 元素，就必须使用全部的元素。它们的出现次序是：thead、tfoot、tbody，这样浏览器就可以在收到所有数据前呈现页脚了。必须在 table 元素内部使用这些标签。

### 框架集标签

```html
        <frameset rows="" cols="">
            <frame src=""/>
            <frame name=""/>
        </frameset>
```

---
## 3 HTML杂项

### HTML标签属性

HTML中的标签有许多属性，用于控制标签的展示样式 或 提供有关 HTML 元素的更多的信息

### HTML注释

`<!-- 注释内容 -->`

### 特殊字符实体

用于展示一些特殊的字符 或 转义html语法使用的字符，具体参考[W3 HTML ISO-8859-1 参考手册](http://www.w3school.com.cn/tags/html_ref_entities.html)

### HTML废弃标签介绍

HTML现在只负责语义，不负责样式，但是HTML初期，也负责标签的样式。经过WEB技术的发展与规范，现在这些样式的标签，都已经被废弃。推荐使用标准的`div+css`构建页面，常用的标签种类`div p h1 span a img ul ol dl input`，

不推荐使用的标签如下：

- 表现性元素：`basefont、big、center、font、s、strike、tt、u`，建议用语义正确的元素代替，并使用CSS来确保渲染后的效果。
- 框架类元素：`frame、frameset、noframes`，因框架有很多可用性及可访问性问题，HTML5规范将这些元素移除。但HTML5支持iframe
- 属性类：
 - align
 - body标签上的link、vlink、alink、text属性
 - bgcolor
 - height和width
 - iframe元素上的scrolling属性
 - valign
 - hspace和vspace
 - table标签上的cellpadding、cellspacing和border属性
 - header标签上的profile属性
 - 链接标签a上的target属性
 - img和iframe元素的longdesc属性
- 其他：`em 、br 、hr 、font 、i 、b`
