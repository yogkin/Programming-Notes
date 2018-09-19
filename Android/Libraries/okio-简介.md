# Okio 学习

---
## 1 Okio的类库介绍

Okio 是 square 公司开发的 `io` 操作类库，它比 java 原始的 io 和 nio 更加方便。Okio 中同样有对应的输入和输出接口，分别是 `Source` 和 `Sink`，`Source`可以看作`OutputStream`，而`Sink`则对应`Sink`。除了常规的操作外，它们还支持读写超时操作。它们同样的都继承了 `Cloaseable` 接口。

### 1.1 `Source`

- `Source` 输入接口
- `BufferedSource` 带缓存功能的输入接口，继承自`Source`
- `RealBufferedSource` 是`BufferedSource`的实现
- `GzipSource` 实现了 `Source` ，提供了Gzip压缩功能
- `InflaterSource`为`GzipSource`提供服务
- `ForwardingSource`一个具有委托功能的抽象类

### 1.2 `Skin`

- `Skin` 输出接口
- `BufferedSkin` 带缓存功能的输出接口，继承自`Skin`
- `RealBufferedSink` 是`BufferedSkin`的实现
- `GzipSkin` 实现了 `Skin` ，提供了Gzip压缩功能
- `DeflaterSkin`为`GzipSkin`提供服务
- `ForwardingSkin`一个具有委托功能的抽象类

`BufferedSink`中定义了一系列写入缓存区的方法，比如write方法写byte数组，writeUtf8写字符串，还有一些列的`writeByte，writeString，writeShort，writeInt，writeLong，writeDecimalLong`等方法；`BufferedSource`定义的方法和BufferedSink极为相似，只不过一个是写一个是读，基本上都是一一对应的，如readUtf8，readByte，readString，readShort，readInt等方法。

### 1.3 `Buffer`

`BufferedSkin`和`BufferedSource`的缓冲区实现都来自一个`Buffer`类，`BufferedSink`通过包装`Buffer`+`Skin`，`BufferedSource`通过包装`Buffer`+`Source`实现。Buffer 通过 Segment 和 SegmentPool 来实现缓存。

### 1.4 `ByteString`

ByteString类，这个类可以用来做各种变化，它将 byte 转换为 String，而这个 String 可以是 utf8 编码的，也可以是 base64 后的值，也可以是 md5 的值，也可以是 sha256 的值。

```
     ByteString string = ByteString.decodeBase64("ZGQ=");//从一个Base64的字符解码
     System.out.println(string.utf8());//输出一个utf-8的字符串
```

### 1.5 `Okio`

`Okio` 相当于一个门面，它有很多的静态方法来创建`Skin`或者`Source`类。

----
## 2 Okio 的使用

`Okio`是门面类，它有很多的静态方法来创建`Skin`或者`Source`类。常用方法如下：

### 输出类

```
    Okio.sink(new File("***"));//从一个文件创建一个Skin
    Okio.sink(new FileOutputStream(new File("***")));//从一个输入流创建一个Skin
    Okio.sink(new Socket("***",8888));//从一个Socket流创建一个Skin
```

### 输入类

```
    Okio.source(new File("***"));//从一个文件创建一个Source
    Okio.source(new FileInputStream(new File("***")));//从一个输入流创建一个Source
    Okio.source(new Socket("****",8888));//从一个Socket流创建一个Source
```

### 缓冲

Okio.buffer()有两个方法，一个用来包装sink，返回一个带缓冲的skin，一个用来包装souce，返回一个带缓存区的source

```
    Okio.buffer(skin)
    Okio.buffer(source)
```

### 原有的内容上添加内容

```
    Sink sink = Okio.appendingSink(file);
```

### NIO

```
    //nio相关
    Okio.sink(path , option)
    Okio.buffer(path , option)
```

---
## 3 示例

写入文件:

```
            BufferedSink bs = null;
            File file = new File(".", "readme02.md");
            try {
                bs = Okio.buffer(Okio.sink(file));
                System.out.println(bs);
                Properties properties = System.getProperties();
                Set<Map.Entry<Object, Object>> entries = properties.entrySet();
                for (Map.Entry<Object, Object> entry : entries) {
                    bs.writeUtf8("key ：" + entry.getKey() + " value：" + entry.getValue());
                    bs.writeUtf8("\r\n");
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (bs != null) {
                    try {
                        bs.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
```

读取内容：

```
            File file = new File(".", "readme02.md");
            BufferedSource source = null;
            try {
                source = Okio.buffer(Okio.source(file));
                String s = source.readUtf8();
                System.out.println(s);
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                if (source != null) {
                    try {
                        source.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
```

---
## 引用

- [Android 善用Okio简化处理I/O操作](http://blog.csdn.net/sbsujjbcy/article/details/50523623)
- [okio解析](http://www.jianshu.com/p/f033a64539a1)
- [拆轮子系列之拆 okio](https://blog.piasy.com/2016/08/04/Understand-Okio/)
- [okio](https://github.com/square/okio)