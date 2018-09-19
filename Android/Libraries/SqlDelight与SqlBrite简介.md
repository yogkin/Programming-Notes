# SqlDelight 与 SqlBrite 简介

---
## 1 SqlDelight

SqlDelight式Square开源的一个数据库效果的库，从SqlDelight的介绍来看其可以根据`CREATE TABLE`sql语句生成java模型接口的库，interface的每个方法就是这张表的每一列。**与ORM的概念相反**

---
## 2 Sqlbrite

Sqlbrite是一个轻量级的封装了`SQLiteOpenHelper`的数据库操作库，主要是把RxJava引入数据操作。其本身并不对sql语句有任何封装

---
## 3 实践感受

1. 使用SqlDelight定义建表语句，然后Gradle插件自动生成抽象的模型类
2. 使用AutoValue实现抽象的模型类相对来说很方便。
3. 使用sql不再有ORM的被束缚感
4. SqlDelight现在并不稳定，每次更新都会有较大的变化，而且根本就没有兼容之前的版本
5. Sqlbrite仅仅式轻量级包装了`SQLiteOpenHelper`，使用RxJava来操作数据库。

---
## 引用

- [sqldelight](https://github.com/square/sqldelight)
- [Android上令人愉快的持久化](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0506/4213.html)
- [delightful-persistence](https://github.com/alexsimo/delightful-persistence)
- [Rxjava+数据库？来用用SqlBrite和SqlDelight吧！](http://blog.csdn.net/u014315849/article/details/51958088)






























