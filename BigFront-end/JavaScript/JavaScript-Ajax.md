# AJAX

---
## 1 什么是Ajax


不用刷新整个页面便可与服务器通讯的办法：

- Flash
- Java applet
- XMLHttpRequest：实际上通常把 Ajax 当成 XMLHttpRequest 对象的代名词 

Ajax即Asynchronous JavaScript and XML。现在，允许浏览器与服务器通信而无须刷新当前页面的技术都被叫做Ajax，Ajax的核心是JavaScript对象XmlHttpRequest。该对象在Internet Explorer 5中首次引入，它是一种支持异步请求的技术。简而言之，XmlHttpRequest使开发者可以使用JavaScript向服务器提出请求并处理响应，而不阻塞用户。

Ajax的运行原理：页面发起请求，会将请求发送给浏览器内核中的Ajax引擎，Ajax引擎会提交请求到服务器端，在这段时间里，客户端可以任意进行任意操作，直到服务器端将数据返回    给Ajax引擎后，会触发你设置的事件，从而执行自定义的js逻辑代码完成某种页面功能。

AJAX的缺陷：AJAX大量使用了Javascript和AJAX引擎，而这个取决于浏览器的支持。IE5.0及以上、Mozilla1.0、NetScape7及以上版本才支持，Mozilla虽然也支持AJAX，但是提供XMLHttpRequest的方式不一样。所以，使用AJAX的程序必须测试针对各个浏览器的兼容性。


---
## 2 JavaScript AJAX

XMLHttpRequest对象：XMLHttpRequest是XMLHTTP组件的对象，通过这个对象，AJAX可以像桌面应用程序一样只同服务器进行数据层面的交换，而不用每次都刷新界面，也不用每次将数据处理的工作都交给服务器来做；这样既减轻了服务器负担又加快了响应速度、缩短了用户等待的时间。

XMLHttpRequest最早是在IE5中以ActiveX组件的形式实现的。非W3C标准，因此创建XMLHttpRequest对象（由于非标准所以实现方法不统一）

 - Internet Explorer把XMLHttpRequest实现为一个ActiveX对象
 - 其他浏览器（Firefox、Safari、Opera…）把它实现为一个本地的JavaScript对象。
 - XMLHttpRequest在不同浏览器上的实现是兼容的，所以可以用同样的方式访问XMLHttpRequest实例的属性和方法，而不论这个实例创建的方法是什么。

```javascript
function   createXmlHttpRequest(){
   var xmlHttp;
   try{    //Firefox, Opera 8.0+, Safari
           xmlHttp=new XMLHttpRequest();
    }catch (e){
           try{    //Internet Explorer
                  xmlHttp=new ActiveXObject("Msxml2.XMLHTTP");
            }catch (e){
                  try{
                          xmlHttp=new ActiveXObject("Microsoft.XMLHTTP");
                  }catch (e){}  
           }
    }
   return xmlHttp;
 }
```

利用XMLHttpRequest 实例与服务器进行通信包含以下3个关键部分:

- onreadystatechange 事件处理函数，监听请求状态与结果
- open 打开请求
- send 发送数据

### onreadystatechange

该事件处理函数由服务器触发，而不是用户，在 Ajax 执行过程中，服务器会通知客户端当前的通信状态。这依靠更新 XMLHttpRequest 对象的 readyState 来实现。改变 readyState 属性是服务器对客户端连接操作的一种方式。每次 readyState 属性的改变都会触发 readystatechange事件。

### open(method, url, asynch)

XMLHttpRequest 对象的 open 方法允许程序员用一个Ajax调用向服务器发送请求。

- method：请求类型，类似 “GET”或”POST”的字符串。若只想从服务器检索一个文件，而不需要发送任何数据，使用GET(可以在GET请求里通过附加在URL上的查询字符串来发送数据，不过数据大小限制为2000个字符)。若需要向服务器发送数据，用POST。
在某些情况下，有些浏览器会把多个XMLHttpRequest请求的结果缓存在同一个URL。如果对每个请求的响应不同，这就会带来不好的结果。把当前时间戳追加到URL的最后，就能确保URL的惟一性，从而避免浏览器缓存结果。 `var url = "GetAndPostExample?timeStamp=" + new Date().getTime(); `
- url：路径字符串，指向你所请求的服务器上的那个文件。可以是绝对路径或相对路径。
- asynch：表示请求是否要异步传输，默认值为true(异步)。指定true，在读取后面的脚本之前，不需要等待服务器的相应。指定false，当脚本处理过程经过这点时，会停下来，一直等到Ajax请求执行完毕再继续执行。

### send(data)

open 方法定义了 Ajax 请求的一些细节。send 方法可为已经待命的请求发送指令。data表示将要传递给服务器的字符串。当向send()方法提供参数时，要确保open()中指定的方法是POST，如果没有数据作为请求体的一部分发送，则使用null。

### setRequestHeader(header,value)

当浏览器向服务器请求页面时，它会伴随这个请求发送一组首部信息。这些首部信息是一系列描述请求的元数据(metadata)。首部信息用来声明一个请求是 GET 还是 POST。

Ajax 请求中，发送首部信息的工作可以由 setRequestHeader完成。比如将 `“Content-type”` 的首部设置为 `“application/x-www-form-urlencoded”`，它会告知服务器正在发送数据，并且数据已经符合URL编码了。该方法必须在open()之后才能调用。

### 处理请求状态

用 XMLHttpRequest 的方法可向服务器发送请求。在 Ajax 处理过程中，XMLHttpRequest 的如下属性可被服务器更改：

- readyState：请求状态
- status：HTTP响应码
- responseText
- responseXML

#### readyState

readyState 属性表示Ajax请求的当前状态。它的值用数字代表。

```
    0 代表未初始化。 还没有调用 open 方法
    1 代表正在加载。 open 方法已被调用，但 send 方法还没有被调用
    2 代表已加载完毕。send 已被调用。请求已经开始
    3 代表交互中。服务器正在发送响应
    4 代表完成。响应发送完毕
```

每次 readyState 值的改变，都会触发 readystatechange 事件。如果把 onreadystatechange 事件处理函数赋给一个函数，那么每次 readyState 值的改变都会引发该函数的执行。
readyState 值的变化会因浏览器的不同而有所差异。但是当请求结束的时候，每个浏览器都会把 readyState 的值统一设为 4

#### responseText

XMLHttpRequest 的 responseText 属性包含了从服务器发送的数据。它是一个HTML,XML或普通文本，这取决于服务器发送的内容。
当 readyState 属性值变成 4 时, responseText 属性才可用，表明 Ajax 请求已经结束。

---
## 3 jQuery的Ajax技术

jquery是一个优秀的js框架，自然对js原生的ajax进行了封装，封装后的ajax的操    作方法更简洁，功能更强大，与ajax操作相关的jquery方法有如下几种，但开发中经常使用的有三种：

```
$.get(url, [data], [callback], [type])
$.post(url, [data], [callback], [type])

    url：代表请求的服务器端地址
    data：代表请求服务器端的数据（可以是key=value形式也可以是json格式）
    callback：表示服务器端成功响应所触发的函数（只有正常成功返回才执行）
    type：表示服务器端返回的数据类型（jquery会根据指定的类型自动类型转换）
    常用的返回类型：text、json、html等


$.ajax( { option1:value1,option2:value2... } );
    async：是否异步，默认是true代表异步
    data：发送到服务器的参数，建议使用json格式
    dataType：服务器端返回的数据类型，常用text和json
    success：成功响应执行的函数，对应的类型是function类型
    type：请求方式，POST/GET
    url：请求服务器端地址
```

---
## 4 安全限制

浏览器的同源策略导致的。默认情况下，JavaScript在发送AJAX请求时，URL的域名必须和当前页面完全一致。

完全一致的意思是，域名要相同（`www.example.com`和`example.com`不同），协议要相同（http和https不同），端口号要相同（默认是`:80`端口，它和`:8080`就不同）。有的浏览器口子松一点，允许端口不同，大多数浏览器都会严格遵守这个限制。

解决方法：

- 通过Flash插件发送HTTP请求，这种方式可以绕过浏览器的安全限制，但必须安装Flash，并且跟Flash交互。
- 通过在同源域名下架设一个代理服务器来转发，JavaScript负责把请求发送到代理服务器
- JSONP

---
## 引用

- [廖雪峰：AJAX](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001434499861493e7c35be5e0864769a2c06afb4754acc6000)
- 同源异源限制
