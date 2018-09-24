# HTML简介

**HTML是负责描述文档语义的语言**，HTML是英语`Hyper Text Markup Language`的缩写，超文本标记语言。`.html`就是网页的格式。

HTML是最基础的网络语言，HTML是通过标签来定义的语言，不区分大小写。**其操作思想是**：为了操作数据,都需要对数据进行不同标签的封装，通过标签中的属性对封装的数据进行操作。标签就相当于一个容器。对容器中的数据进行操作，就是在不断地改变容器的属性值

现在的业界的标准，网页技术严格的三层分离：

- **html**就是负责描述页面的语义；
- **css**负责描述页面的样式；
- **js**负责描述页面的动态效果的。

所以，html不能让文字居中，不能更改文字字号、字体、颜色。因为这些都是属于样式范畴，都是css干的事；html不能让盒子运动起来，因为这些属性行为范畴，都是js干的事。html只能干一件事，就是通过标签对，给文本增加语义。这是html唯一能做的.**html中，除了语义，其他什么都没有**

---
## 1 HTML骨架

### 完整的骨架

```html
        <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
        <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">

            <head>
                <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
                <title>哈哈哈</title>
            </head>

            <body>
                 <h1>我是一个主标题</h1>
                 <p>我是一个小段落</p>
            </body>

        </html>
```

### 文档声明头

第1行是网页的**声明头**，网页声明头可以告诉浏览器，这是一个什么标准的页面。术语叫做**DocType Defintion**，文档类型定义，简称DTD。

**W3C**就是出web规范的组织机构。html、css、js的规范都是W3C定义发布的。W3C即`world wide web coalition `国际万维网联盟。

任何一个标准的HTML页面，第一行一定是一个以`<!DOCTYPE ……`开头的语句。此标签可告知浏览器文档使用哪种 HTML 或 XHTML 规范。那么到底有哪些规范呢？


---
## 2 Html版本与规范

我们现在学习的是**HTML4.01**这个版本，这个版本是IE6开始兼容的。HTML5是IE9开开始兼容的。但是IE6、7、8这些浏览器还不能过早的淘汰，所以这几年网页还是应该用HTML4.01来制作。


**HTML4.01**里面有两大种规范，每大种规范里面又各有3种小规范。所以一共6种规范，`HTML4.01`里面规定了**普通、XHTML**两大种规范。

>HTML觉得自己有一些规定不严谨，比如，标签是否可以用大写字母呢？`<H1></H1>`所以，HTML就觉得，把一些规范严格的标准，又制定了一个XHTML1.0。在XHTML中的字母X，表示“严格的”。

总结一下，一共有6种DTD，说白了，HTML第一行语句一共有6种：

大规范|小规范
---|---
HTML 4.01| Strict 严格的，体现在一些标签不能使用，比如`u`
HTML 4.01| Transitional   普通的
HTML 4.01| Frameset     带有框架的页面
XHTML1.0严格体现在小写标签、闭合、引号|Strict 严格的，体现在一些标签不能使用，比如`u`
XHTML1.0严格体现在小写标签、闭合、引号|Transitional   普通的
XHTML1.0严格体现在小写标签、闭合、引号|Frameset     带有框架的页面

- **strict**表示**严格的**，这种模式里面的要求更为严格,这种严格体现在哪里？有一些标签不能使用，比如`u`标签，就是可以让一个本文加上下划线，但是这和HTML的本质有冲突，因为HTML只能负责语义，不能负责样式，而u这个下划线是样式。所以，在strict中是不能使用u标签的。`<u>嘻嘻嘻嘻嘻</u>`怎么给文本增加下划线呢？使用css属性来解决。
- **Transitional**表示“普通的”，这种模式就是没有一些别的规范
- **Frameset**表示“框架”，在框架的页面使用


**而Html5的文档声明却很简单**：

    <!DOCTYPE html>

---
## 3 meta

meta表示**元**。**元配置**，就是表示基本的配置项目

### 3.1 字符集

字符集用meta标签定义：

```
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
```

中文能够使用的字符集两种:

```
    第一种：UTF-8
        <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    第二种：gb2312
        <meta http-equiv="Content-Type" content="text/html;charset=gb2312">
    也可以写成gbk(不推荐)
        <meta http-equiv="Content-Type" content="text/html;charset=gbk">
```

>http-equiv，相当于http的文件头作用，它可以向浏览器传回一些有用的信息，以帮助正确和精确地显示网页内容


**我们用meta标签可以声明当前这个html文档的字库，但是一定要和保存的类型一样，否则乱码！**

两种字符集的优缺点

- UTF-8 字多，有各种国家的语言，但是保存尺寸大，文件臃肿；
- gb2312字少，只用中文和少数外语和符号，但是尺寸小，文件小巧。

列出2个使用情形：

- 你们公司是做日本动漫的，经常出现一些日语动漫的名字，网页要使用UTF-8。如果用gb2312将无法显示日语。
- 你们公司就是中文网页，极度的追求网页的显示速度，要使用gb2312。如果使用UTF-8将每个汉字多一个byte，所以5000个汉字，多5kb。

### 3.2 关键字和页面描述

meta除了可以设置字符集，还可以设置关键字和页面描述。

#### 设置页面描述

```
    <meta name="Description" content="网易是中国领先的互联网技术公司，为用户提供免费邮箱、游戏、搜索引擎服务，开设新闻、娱乐、体育等30多个内容频道，及博客、视频、论坛等互动交流，网聚人的力量。" />
```

只要设置的Description页面面熟，那么百度搜索结果，就能够显示这些语句，**这个技术叫做SEO，search engine optimization**，搜索引擎优化。

#### 定义关键词

```
    <meta name="Keywords" content="网易,邮箱,游戏,新闻,体育,娱乐,女性,亚运,论坛,短信" />
```
这些关键词，就是告诉搜索引擎，这个网页是干嘛的，能够提高搜索命中率。让别人能够找到你，搜索到你。Keywords就是“关键词”的意思。

---
##  4 常用标签

常用参考`w3c school`

HTML标签可以分为**容器级的标签**和**文本级的标签**。

它们的区别在于：

*   容器级的标签中可以嵌套其它所有的标签
*   常见容器级的标签: `div h ul ol dl li dt dd ...`
*   文本级的标签中只能**嵌套文字/图片/超链接**
*   常见文本级的标签:`span p buis strong em ins del ...`
