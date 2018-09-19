# 第六章：使用集合类

---
## 1 List

```groovy
list = [1,2,3,4,5,6,7,8,9]
println list
println list.dump()
println list.getClass().name//java.util.ArrayList
println list[0]// 1
println list[-1]//倒着取 9
println list[1..3]//[2, 3, 4]
println list[-2..0]//倒着取 [8, 7, 6, 5, 4, 3, 2, 1]


//查看list[1..3]返回的类型是什么
subList = list[1..4]
println subList.getClass().name
println subList.dump()//<java.util.ArrayList@f0c03 elementData=[2, 3, 4, 5] size=4 modCount=1> 可见返回的是一个新的子对象
```

### 迭代List:

迭代器：
外部迭代器：比如for循环，可以控制迭代
内部迭代器：在支持闭包的语言中很流行，不能控制迭代，但更容易使用

```groovy
list.each {
    print it+" "
}
println ""
list.reverseEach {//反向迭代
    print it+" "
}
list.eachWithIndex { int entry, int i ->
    println "entry is $entry , index is $i"
}
```

### 对闭包中的元素求和
```groovy
total = 0
list.each {
    total += it
}
println total
//把集合中的每个元素变成原来的两倍
doubleList = []
list.each {
    doubleList << 2 * it
}
println doubleList
```

### 使用List的collect方法

collect方法对每个元素应用一次变换，然后再把变换后的元素收集为一个新的集合
```groovy
println list.collect( ){it*2}//结果为：[2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### 使用查找方法

```groovy
println list.find(){//返回第一个满足条件的元素
    it >5
}
println list.findAll(){//返回一个集合，满足条件的所有元素
    it > 6
}
println list.findIndexOf {
    it > 2
}
```
### List上的其他便捷方法

- each
- collect
- sum
- join
- flatten
- minus
- reverse
- inject

```groovy
//each方法用于遍历元素
list1 = ["Programming" , "in"  , ' Groovy']
def count  = 0
list1.each {
    count += it.size()
}
println count

//collect{it.size()} 返回每个元素的count， sum用于求和
println list1.collect{it.size()}.sum()

//inject接收要给初始值carryOver，把carryOver和每一个元素作为闭包的参数，并获取闭包返回的值，下次调用闭包时，传入此值
println list1.inject(0){
    carryOver, element ->
        carryOver + element.size()
}

//join 用于连接每个元素
println list1.join("")
println list1.join("----")

// flatten用于拉平一个list
list2 = [list1, ' aaaaaaaa', 'bbbbbb']
println list2.flatten()//[Programming, in,  Groovy,  aaaaaaaa, bbbbbb]

// 使用minus()方法(映射到的是-操作符)
println list1 - ['in']//返回的是一个新的String

//得到一个反序的副本
println list.reverse()
```
### 展开操作符

展开操作符(*.)通常被集合对象调用其所有的对象使用。等价于遍历调用,并将结果收集在一个集合中

```groovy
println list.size()
println list1*.size()//结果为：[11, 2, 7] 因为*的影响，即作用于列表的每一个元素，变为展开操作符， 相当于下面代码

println list1.collect{
    it.size()
}


println list*.equals("")//[false, false, false, false, false, false, false, false, false]


//*的应用
def printWord(a, b, c) {
    println "$a $b $c"
}
printWord(//必须保证参数个数一致
        *list1
)
```


---
## 2 Map

```groovy
//定义Map的方式
langs = [
        'c++':'Stroustrup',
        'java':'Gosling',
        'lisp':'McCarthy'
]

println langs.getClass().name//结果：java.util.LinkedHashMap ，使用getClass，而不是直接访问class，因为Groovy会认为你是根据key取value
println langs."c++"//因为+的原因，加上“”
println "langs.java:"+langs.java
println "langs['java']:"+langs['java']

//定义map时，也可以不带引号，(因为key必须为String)
langs = [
        'c++':'Stroustrup',
        java:'Gosling',
        lisp:'McCarthy'
]
```
### 在Map上迭代

```groovy
langs.each {//一个参数的闭包
    Map.Entry entry->
        println entry.key+"   "+entry.value
}
langs.each {//两个参数的闭包
    key,value->
        println key+"   "+value
}

### Map的collect方法

println langs.collect {//返回一个新的List
    key,value->
        key.replaceAll("[+]","p")//此处返回的一个String
}
```

### find和findAll方法

```groovy
println "===========find findAll ================"
println langs.find {//返回的是LinkedHashMap$Entry
    key,value->
        key.size() > 3
}
println langs.findAll {//返回一个新的map
    key,value->
        key.size()>2
}
```

### Map上的其他便捷方法

```groovy
println "=========== Map上的其他便捷方法 ================"

println langs.any {//用于确定map中有任何元素满足条件
    key,value->
        (key == "java")
}
println langs.every {//用于确定map中所有元素满足条件
    key,value->
    key.size()>3
}

friends = [
        briang:"Brian Goetz",
        brians:"Brian Sletten",
        dividb:"Divid Block"
]

def map = friends.groupBy {//根据返回一个map
    it.value.split(' ')[0]
}//[Brian:[briang:Brian Goetz, brians:Brian Sletten], Divid:[dividb:Divid Block]]
println map
```

**最后：Groovy的map用于具名参数，以及使用Map实现接口**


---
## 3 Range

range 是对List的一种扩展，变量定义和大体的使用方法如下：

```groovy
        def aRange  = 1..3   // range类型有begin值 加两个点 +end 值表示  左边这aRange包含 1 , 2 , 3 .这三个值

        println("aRange = " + aRange)
        //如果 不想包含最后一个元素，则可以使用<，
        def aRange2 = 1..<10
        println("aRange2 = " + aRange2)
        println(aRange.from)
        println(aRange.getFrom())
        println(aRange.to)
```

