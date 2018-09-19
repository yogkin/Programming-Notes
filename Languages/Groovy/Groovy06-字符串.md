# 第五章：使用字符串

---
## 1 Groogy字符串基础

Groovy中可以使用单引号 创建字符串，比如'hello',在java中'a'是一个char，"a"才是一个String，
但是在Groovy中并没有这样的区别，如果需要创建一个char，可以使用` a = 'a' as char`

```groovy
//双引号都可以直接放在字符串中，而不需要使用转义字符
println 'he said , "that is Groovy"'
str = 'string'

println str.getClass().name//输出：java.lang.String
/*
从输出看，''创建的字符串是java中的String，单引号的字符串，Groovy会直接转换成java中的String，而不会对其进行运算，如果需要运算，需要使用双引号字符串
*/

val1 = 32
println "the val1 is $val1"//输出：the val1 is 32
//从输出看以看出Groovy对其进行了运算，

//可以使用[]操作符来操作读取字符串中对应位置的字符串，但是不能修改，因为String是不可变对象
println str[1]
```

使用`双引号`和`//`都可以创建字符串，双引号常用于定义字符串表达式，而`/`用于正则表达式


```groovy
val2 = 12;
println "he paid \$${val2} for that."//这里使用了\来转义字符$

//使用/则不需要转义
println(/he paid $$val2 for that. /)
```

对于字符串中的$求值，简单的变量名和属性读取可以直接使用` $value`方式，而如果是复杂的求值需要使用`${表达式}`

---
## 2 GString的惰性求值


```groovy

what = new StringBuilder('fence')
text = "the cow jumped over the $what"
println  text
what.replace(0,5,"moon")//惰性求值
println text

//查看GString的真实实现
def printClassInfo(obj) {
    println "class : ${obj.getClass().name}"
    println "class: ${obj.getClass().superclass.name}"
}
val3 = 124;
printClassInfo("the stock closed at $val3")
printClassInfo(/the stock closed at $val3/)
printClassInfo("this is a String")
```
打印结果为：

class : org.codehaus.groovy.runtime.GStringImpl
class: groovy.lang.GString
class : org.codehaus.groovy.runtime.GStringImpl
class: groovy.lang.GString
class : java.lang.String
class: java.lang.Object

**Groovy并不会简单的因为使用了双引号或者正斜杠就使用GString，而是会智能的选择**

GString的惰性求值问题：上面`what = new StringBuilder('fence')`的实例显得相当的合理，但是如果修改的是what的引用则情况就不一样的

```groovy
price = 684.71
def company = 'google'
quote = "today $company stock closed at $price"
println quote
stocks = [Apple: 663.01, Microsoft:30.95]
stocks.each {
    key,value->
        company = key
        price = value
        println quote
}
```
打印结果是：

today google stock closed at 684.71
today google stock closed at 684.71
today google stock closed at 684.71

可见，虽然改变了comany和price的引用，但是打印的语句却没有改变,可见可以改变comany和price的引用，却不能改变GString绑定的内容，**使用闭包来解决问题**：

```groovy
companyClosure = {
    it.write(company)
}
priceClosure = {
    it.write("$price")
}
quote1 = "today ${companyClosure} stock closed at ${priceClosure}"
stocks.each {
    key,value ->
        company = key
        price = value
        println quote1
}
```
打印结果是：

today Apple stock closed at 663.01
today Microsoft stock closed at 30.95

原理是：但对GString进行求值时，如果其中包含一个变量，该变量会被简单的打印到Writer，通常是StringWriter，
如果GSting包含的是一个闭包，该闭包就会被调用，传入的是这个StringWriter,如果闭包不接受参数，则不传入参数，但是如果闭包接受的参数超过一个，就会抛出异常。

去掉it参数，GString会使用闭包返回的参数

```groovy
companyClosure = {->
     company
}
priceClosure = {->
    price
}
quote2 = "today ${companyClosure} stock closed at ${priceClosure}"
stocks.each {
    key,value ->
        company = key
        price = value
        println quote2
}

//更加简单的方法
quote3 = "today ${->company} stock closed at ${->price}"
stocks.each {
    key,value ->
        company = key
        price = value
        println quote3
}
```

---
## 3 多行字符串

```groovy
memo = '''
            你好啊
            我在学习Groovy
'''
println memo
val1  = 234000
//"""同样支持求值
memo1 = """

                        我在学习Groovy
                        GString挺不错的
              \$ ${val1}

"""
println memo1


//处理xml
langs = ['c++':'Stroustrup', 'java':'Gosling']

content = ''
langs.each {
    key,value->
        fragment = """
            <language name="$key">
                <author>$value</author>
            </language>
        """
        content+=fragment

}
println content
```

---
## 4 字符串便捷的方法

String的execute方法可以帮我们创建一个process对象，他还有其他重载的操作符

```groovy
str = "It is a rainy day in seattle"
println str
str -= "rainy"
println str
```

打印结果：
It is a rainy day in seattle
It is a  day in seattle

`-=`用于从原有的字符串中减去字符串，其他的操作符还有` plus + ， multiply * , next ++ , replaceAll和tokenize`等

**还可以迭代String**:
```groovy
for(str1 in 'held'.."helm"){
    print str1+" "
}//held hele helf helg helh heli helj helk hell helm  这里映射的是next操作符

println 'a'.next()
```

**索引String**:
```groovy
str3 = "abcdefg"
println "str3[2..5] = ${str3[2..5]}"
```

---
## 5 正则表达式

```groovy
obj = ~"hello"//~用于方便的创建正则表达式
println obj.getClass().name//java.util.regex.Pattern

//单/双引号和正斜杠都可以创建字符串，而正斜杠不需要转义
//=~和==~是Groovy提供的操作符，

pattern = ~"(G|g)roovy"
text = 'Groovy is hip'

if (text =~ pattern) {
    println "match"
}else {
    println "no match"
}

if (text ==~ pattern) {
    println "match"
}else {
    println "no match"
}
```
结果：
match
no match
=~  执行部分匹配
==~ 执行精确匹配

`=~`返回一个Matcher读写，`java.util.regex.Matcher`实例，Matcher的boolean求值实现是至少有一个匹配，就会返回true，如果有多个匹配，则matcher会包含一个匹配的数组：

```groovy
matcher = "Groovy is groovy" =~ /(G|g)roovy/
println "size of matcher is ${matcher.size()}"
println "with elements ${matcher[0]} and ${matcher[1]}"
```

可以使用replaceFirst方法或者replaceAll方法方便的替换匹配文本：

```groovy
str2 = 'groovy is groovy ,really groovy'
println str2
result = (str2 =~/groovy/).replaceAll("hip")
println result
```

总结：
- 要从字符串创建一个模式，使用`~`操作符
- 要定义一个RegEx，使用正斜杠，向`/[G|g]roovy/`
- 要确定是否存在匹配，使用`=~`
- 对于精确匹配，使用`==~`

