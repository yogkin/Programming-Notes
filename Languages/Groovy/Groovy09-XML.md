# 第八章：处理xml

预备xml

```xml
test1.xml:

<?xml version="1.0" encoding="utf-8"?>
<languages>

    <language name="c++">
        <author>Stroustrup</author>
    </language>
    <language name="java">
        <author>Gosling</author>
    </language>
    <language name="Lisp">
        <author>McCarthy</author>
    </language>

</languages>

test2.xml:

<?xml version="1.0" encoding="utf-8"?>
<languages
    xmlns:computer="computer" xmlns:natural="natural">

    <computer:language name="c++"/>
    <computer:language name="java"/>
    <computer:language name="Lisp"/>

    <natural:language name="English"/>
    <natural:language name="Chinese"/>
    <natural:language name="French"/>

</languages>
```
---
## 1 解析xml

使用Groovy的分类可以在类上定义动态的方法，其中一个分类`DOMCategory(DOM:Document Object Model)`可以处理文档对象模型DOMCategory可以通过`GPath(Groovy path extension)`：


```groovy
document = DOMBuilder.parse(new FileReader('test1.xml'))
rootElement = document.documentElement//这里的rootElement并不是languages

println rootElement.childNodes.length
println rootElement.getClass().name

use(DOMCategory){
    println "language and authors "
    languages = rootElement.language//而languages = rootElement.language 可以理解为从rootElement获取所有的language，即languages
    println languages.author//[[author: null], [author: null], [author: null]] 从打印的信息可以看出 languages代表的是所有的language

    languages.each {
        language->
        println "${language.'@name'} authored by ${language.author[0].text()}"//'@name'语法用于访问属性，author[0]表示访问第一个子节点，text()表示获取节点里的内容
    }

    def languageByAuthor = {
        authorName ->
            languages.findAll{it.author[0].text() == authorName}.collect{
                it.'@name'
            }.join(' , ')//获取所有作者为authorName的 书名的文本
    }
    println "Language By Gosling : "+languageByAuthor('Gosling')
}
```

DOMCategory结合GPath可以很方便的处理xml,要使用DOMCategory必须把代码放在use块内。

**GPath是什么**：
与XPath类似，GPath可以帮助导航对象Plain( Old Java Object 普通java对象)和( Old Groovy Object)和xml层次结构使用`.`可以遍历层次结构， 例如`car.engine.power`这种写法会帮助我们通过要给car实例的getEngine方法访问engine属性，然后通过或取得的engine实例的getPowder方法方法该实例的powder属性即` . `用来获取子元素而` car.‘@year'` 或者` car.@year `用于获取car节点上的year属性，而不是子节点。

---
## 2 使用XMLParser

使用XMLParser,利用Groovy的动态类型和元编程能力，可以直接使用名字访问文档中的成员，例如使用it.author[0]访问一个作者的名字

```groovy
languages = new XmlParser().parse('test1.xml')
println "Language and Author"
languages.each {
    println "${it.@name} author by ${it.author[0].text()}"
}

def languageByAuthor = {
    authorName->
        languages.findAll{
            it.author[0].text() == authorName
        }.collect{
            it.@name
        }.join(' , ')
}

println languageByAuthor("Stroustrup")
```

---
## 3 使用XmlSlurper

对于较大的文档，使用XmlParser对内存的压力是比较大的，而使用XmlSlurper可以处理较大的xml
其使用上与XmlParser类似

```groovy
languages = new XmlSlurper().parse('test1.xml')
languages.language.each {//还是有一定的区别的，要先通过.language获取所有的language
    println "${it.@name} author by ${it.author[0].text()}"
}

//处理命名空间
languagesNs = new XmlSlurper().parse('test2.xml')
.declareNamespace(human:'natural')//declareNamespace方法用于声明一个命名空间，语法为    human:'natural' 其中human可以随便取，用于标识natural命名空间
println languagesNs.language.collect{it.@name}.join(' , ')
println languagesNs.'human:language'.collect{it.@name}.join(' , ')//.'human:language'用于获取human对应命名空间的所有元素
```

---
## 4 创建xml

1. 使用GString的嵌入能力创建xml前面 已经学习过了
2. 使用MarkupBuilder或者StreamingMarkupBuilder示例：(具体参见第17.1章)

```groovy
langs = ['c++':'Stroustrup' ,'java':'Gosling', 'Lisp':'McCarthy']

/*
 that mkp is a special namespace used to escape away from the normal building mode of the builder
 and get access to helper markup methods 'yield', 'pi', 'comment', 'out', 'namespaces', 'xmlDeclaration' and 'yieldUnescaped'.
 Note that tab, newline and carriage return characters are escaped within attributes, i.e. will become and respectively
 */
xmlDocument = new StreamingMarkupBuilder().bind {
    println mkp//.BaseMarkupBuilder$Document@4690b489 mkp是一个特殊的命名，指向Document
    mkp.xmlDeclaration()//声明xml 文件 <?xml version='1.0'?>
    mkp.declareNamespace(computer:"Computer")//声明命名空间
    languages{//声明languages节点
        comment  << "Created using StreamingMarkupBuilder"//注释
        langs.each{
            key,value->
                computer.language(name:key){//使用computer命名空间 创建language节点，并设置属性name为key
                    author(value)//在language节点在创建author节点，把value作为其内容
                }
        }
    }
}

println xmlDocument
new File('test3.xml').withWriter {
    it << xmlDocument
}
```

结果：


test3.xml:
```xml
<?xml version='1.0'?>
<languages xmlns:computer='Computer'>
    <!--Created using StreamingMarkupBuilder-->
    <computer:language name='c++'>
        <author>Stroustrup</author>
    </computer:language>
    <computer:language name='java'>
        <author>Gosling</author>
    </computer:language>
    <computer:language name='Lisp'>
        <author>McCarthy</author>
    </computer:language>
</languages>
```