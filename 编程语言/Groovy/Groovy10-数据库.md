# 第九章：使用数据库

预备：

-  本地安装mysql
-  类库中加入`my-mysql-connector.jar`包

准备数据库：

```SQL
create database if not exists weatherinfo;
use weatherinfo;
drop table if exists weather;
create table weather(
        city varchar(100) not null,
        temperature integer not null
);
insert into weather (city,temperature) values ('Austin', 48);
insert into weather (city,temperature) values ('Boton Rouge', 21);
insert into weather (city,temperature) values ('Jsckson', 2);
insert into weather (city,temperature) values ('Montgomery', 32);
insert into weather (city,temperature) values ('Sacramento', 22);
insert into weather (city,temperature) values ('Santa', 1);
insert into weather (city,temperature) values ('Tallahassee', 23);
insert into weather (city,temperature) values ('Shenzhen', 17);
```
---
## 1 使用Groovy连接数据库

```
def sql = Sql.newInstance('jdbc:mysql://localhost:3306/weatherinfo', 'root', '201314', 'com.mysql.jdbc.Driver')
println sql.connection.catalog
//select操作
println "city                   temperature"
sql.eachRow('SELECT * FROM  weather') {
    printf " %s        %s   \n", it.city, it[1]
}
```

eachRow的另外要给重载，接受两个闭包，一个用于元数据，另一个用于数据，元数据的闭包调用一次
并以一个ResultSetMetaData实例为参数，而另一个闭包会对结果中的每一行都调用一次

```groovy
processMeta = {  metaData ->
    metaData.columnCount.times{
        i ->
            printf "%s      " , metaData.getColumnLabel(i + 1)
    }
    println ''
}

sql.eachRow('SELECT * FROM  weather', processMeta){
    printf " %s        %s   \n", it.city, it[1]
}

//使用rows方法返一个数据的结果集ArrayList
rows = sql.rows('SELECT * FROM weather')
println  rows.size()
//使用firstRow仅返回第一行
//可以使用sql的call方法执行存储过程，使用withStatement方法可以设置一个将在查询之前调用的闭包，如果想在执行之前拦截
//并修改Sql查询，该方法会有所帮助。
sql.close()//记得关闭sql
```

---
## 2 将数据转换为xml
```groovy
def sql = Sql.newInstance('jdbc:mysql://localhost:3306/weatherinfo', 'root', '201314', 'com.mysql.jdbc.Driver')
bldr = new MarkupBuilder()
bldr.weather{
    sql.eachRow('SELECT * FROM  weather'){
        city(name:it.city,temperature:it.temperature)
    }
}
sql.close()
```

---
##  3 使用DataSet

DataSet表示一个查询后的结果集，sql的dataSet方法返回哟个虚拟代理，知道迭代时才去实际获取数据

```
def sql = Sql.newInstance('jdbc:mysql://localhost:3306/weatherinfo', 'root', '201314', 'com.mysql.jdbc.Driver')
dataSet = sql.dataSet('weather')
println dataSet.each {
    println it
}
println dataSet.rows()
```

### 插入数据

**使用dataSet来插入数据**：

```
dataSet.add(city:"ChangSha",temperature:3)
```

结果：
```
mysql> select * from weather;
+-------------+-------------+
| city        | temperature |
+-------------+-------------+
| Austin      |          48 |
| Boton Rouge |          21 |
| Jsckson     |           2 |
| Montgomery  |          32 |
| Sacramento  |          22 |
| Santa       |           1 |
| Tallahassee |          23 |
| Shenzhen    |          17 |
| ChangSha    |           3 |
+-------------+-------------+
```

**使用sql来插入数据**

```
sql.execute(
        """INSERT INTO  weather VALUES ('haha',100)"""
)
```
结果：
```
mysql> select * from weather;
+-------------+-------------+
| city        | temperature |
+-------------+-------------+
| Austin      |          48 |
| Boton Rouge |          21 |
| Jsckson     |           2 |
| Montgomery  |          32 |
| Sacramento  |          22 |
| Santa       |           1 |
| Tallahassee |          23 |
| Shenzhen    |          17 |
| ChangSha    |           3 |
| haha        |         100 |
+-------------+-------------+
```