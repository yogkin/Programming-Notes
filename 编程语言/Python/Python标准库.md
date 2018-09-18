# Python 标准库

Python有一套很有用的标准库(standard library)。标准库会随着Python解释器，一起安装在你的电脑中的。 它是Python的一个组成部分。这些标准库是Python为你准备好的利器，可以让编程事半功倍。

---
## 常用标准库

| 标准库 | 说明 |
| --- | --- |
| builtins | 内建函数默认加载 |
| os | 操作系统接口 |
| sys | Python自身的运行环境 |
| functools | 常用的工具 |
| json | 编码和解码 JSON 对象 |
| logging | 记录日志，调试 |
| multiprocessing | 多进程 |
| threading | 多线程 |
| copy | 拷贝：深拷贝(copy)、深拷贝(deepcopy) 不仅复制对象本身，同时也复制该对象所引用的对象|
| time | 时间：sleep |
| datetime | 日期和时间 |
| calendar | 日历 |
| hashlib | 加密算法 |
| random | 生成随机数 |
| re | 字符串正则匹配 |
| socket | 标准的 BSD Sockets API |
| shutil | 文件和目录管理 |
| glob | 基于文件通配符搜索 |
| enum | 枚举 |
| functools | functools 是python2.5被引人的,一些工具函数放在此包里。 |
| types | types模块定义了python中所有的类型 |

---
## 常用扩展库

| 扩展库 | 说明 |
| --- | --- |
| requests | 使用的是 urllib3，继承了urllib2的所有特性 |
| urllib | 基于http的高层库 |
| scrapy | 爬虫 |
| beautifulsoup4 | HTML/XML的解析器 |
| celery | 分布式任务调度模块 |
| redis | 缓存 |
| Pillow(PIL) | 图像处理 |
| xlsxwriter | 仅写excle功能,支持xlsx |
| xlwt | 仅写excle功能,支持xls ,2013或更早版office |
| xlrd | 仅读excle功能 |
| elasticsearch | 全文搜索引擎 |
| pymysql | 数据库连接库 |
| mongoengine/pymongo | mongodbpython接口 |
| matplotlib | 画图 |
| numpy/scipy | 科学计算 |
| django/tornado/flask | web框架 |
| xmltodict | xml 转 dict |
| SimpleHTTPServer | 简单地HTTP Server,不使用Web框架 |
| gevent | 基于协程的Python网络库 |
| fabric | 系统管理 |
| pandas | 数据处理库 |
| scikit-learn | 机器学习库 |


--- 
## functools

### partial函数(偏函数)

把一个函数的某些参数设置默认值，返回一个新的函数，调用这个新函数会更简单。

```python
import functools

def showarg(*args, **kw):
    print(args)
    print(kw)

p = functools.partial(showarg, 1,2,3)
```

#### wraps函数

使用装饰器时，有一些细节需要被注意。例如，被装饰后的函数其实已经是另外一个函数了（函数名等函数属性会发生改变）。Python的functools包中提供了一个叫wraps的装饰器来消除这样的副作用

---
## 内置函数

- dir：函数不带参数时，返回当前范围内的变量、方法和定义的类型列表；带参数时，返回参数的属性、方法列表。如果参数包含方法`__dir__()`，该方法将被调用。如果参数不包含`__dir__()`，该方法将最大限度地收集参数信息。
- next：返回迭代器的下一个项目。
- len：获取对象长度。
- hasattr：对象是否存在某个属性。
- getattr：获取对象的属性。
- setattr：给对象设置属性。
- delattr：删除对象的属性。
- range：创建一个range。
- map：map函数会根据提供的函数对指定序列做映射。
- filter：filter函数会对指定序列执行过滤操作。
- reduce：reduce函数会对参数序列中元素进行累积。
- sorted：用于排序。
- eval：执行python代码

Python内置了许多有用的函数，具体参考：

- [Built-in Functions](https://docs.python.org/3/library/functions.html)
- [Python3内置函数](http://www.runoob.com/python3/python3-built-in-functions.html)


---
## 引用

- [Python 3 标准库](http://docspy3zh.readthedocs.io/en/latest/library/)
- [Python 3 标准库文档](https://docs.python.org/3/library/index.html)
