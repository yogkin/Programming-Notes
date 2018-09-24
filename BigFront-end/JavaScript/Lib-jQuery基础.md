# jQuery基础

---
## 1 简介

为了简化 JavaScript 的开发, 一些 JavsScript 库诞生了. JavaScript 库封装了很多预定义的对象和实用函数。能帮助使用者建立有高难度交互的 Web2.0 特性的富客户端页面, 并且兼容各大浏览器。

jQuery由美国人John Resig创建，是继prototype之后又一个优秀的Javascript框架。其宗旨是——`WRITE LESS,DO MORE`，写更少的代码，做更多的事情。jQuery是一个快速的，简洁的javaScript库，使用户能更方便地处理HTML documents、events、实现动画效果，并且方便地为网站提供AJAX交互。jQuery能够使用户的html页保持代码和html内容分离，也就是说，不用再在html里面插入一堆js来调用命令了，只需定义id即可。

### jQuery库包含以的功能

*   HTML 元素选取
*   HTML 元素操作
*   CSS 操作
*   HTML 事件函数
*   JavaScript 特效和动画
*   HTML DOM 遍历和修改
*   AJAX
*   Utilities
*   除此之外，Jquery还提供了大量的插件。

### jQuery版本

有两个版本的 jQuery 可供下载：

*   Production version - 用于实际的网站中，已被精简和压缩。
*   Development version - 用于测试和开发（未压缩，是可读的代码）

注意，jQuery的2.x版本后，不再支持IE6、7、8。

### jQuery解决了什么问题

如果手写js，我们可能遇到下面问题：

1.    window.onload 事件有个事件覆盖的问题，我们只能写一个
2.    代码容错性差
3.    浏览器兼容性问题
4.    书写很繁琐，代码量多
5.    代码很乱，各个页面到处都是
6.    动画效果，实现比较繁琐

---
## 2 jQuery入门与入口函数

### 基本语法

- jQUery的两个全局变量：`$ 和 jQuery`，这两个变量是完全相等的。

基本语法：

```javascript
$(selector).action();
```

### 入口函数

1. Js的`window.onload`事件是等到所有内容(包括外部图片、css、js等文件)加载完了之后，才会去执行
2. jQuery的入口函数 是在html所有标签都加载之后，就会去执行。

```javascript
//方式1
jQuery(document).read(function(){  });
//方式2
$(function(){ });
```

---
## 3 选择器

jQuery选择器有：基本选择器、层级选择器、基本过滤选择器、属性选择器、表单选择器。jQuery 中所有选择器都以美元符号开头：`$()`。

### 基本选择器

符号 | 说明 | 用法
---|---|---
`$(“#demo”)` | 选择id为demo的第一个元素 | `$(“#demo”).css(“background”,”red”)`
`$(“.liItem”)` | 选择所有类名（样式名）为liItem的元素 | `$(“.liItem”). css(“background”,”red”);`
`$(“div”)` | 选择所有标签名字为div的元素 | `$(“div”). css(“background”,”red”);`
`$(“*”)` | 选择所有元素少用或配合其他选择器来使用 | `$(“*”).css(“background”,”red”)`
`$(“.liItem,div”)` | 选择多个指定的元素，这个地方是选择出了 .liItem元素和div元素 | `$(“.liItem,div”). css(“background”,”red”)`

### 层级选择器

符号 | 说明 | 用法
---|---|---
`空格` |后代选择器 选择所有的后代元素 | `$(“div span”). css(“background”,”red”);`
`>` |子代选择器 选择所有的子代元素 | `$(“div > span”). css(“background”,”red”)`
`+` |紧邻选择器 选择紧挨着的下一个元素 | `$(“div + p”). css(“background”,”red”)`
`~` |兄弟选择器 选择后面的所有的兄弟元素 | `$(“div ~ p”). css(“background”,”red”)`


### 过滤选择器

符号 | 说明 | 用法
---|---|---
`:eq(index)` | index是从0开始的一个数字，选择序号为index的元素。选择第一个匹配的元素。 | `$(“li:eq(1)”). css(“background”,”red”)`
`:gt(index)` | Index是从0开始的一个数字，选择序号大于index的元素 | `$(“li:gt(2)”). css(“background”,”red”)`
`:lt(index)` | Index是从0开始的一个数字，选择小于index 的元素 | `$(“li:lt(2)”). css(“background”,”red”)`
`:odd` | 选择所有序号为奇数行的元素 | `$(“li:odd”). css(“background”,”red”)`
`:even` | 选择所有序号为偶数的元素 | `$(“li:even”). css(“background”,”red”)`
`:first` | 选择匹配第一个元素 | `$(“li:first”). css(“background”,”red”)`
`:last` | 选择匹配的最后一个元素 | `$(“li:last”). css(“background”,”red”)`

### 属性选择器

符号 | 说明 | 用法
---|---|---
`$(“a[href]”)` | 选择所有包含href属性的元素 | `$(“a[href]”). css(“background”,”red”)`
`$(“a[href=’itcast’]”)` | 选择href属性值为itcast的所有a标签 | `$(“a[href=’itcast’]”). css(“background”,”red”)`
`$(“a[href!=’baidu’]”)` | 选择所有href属性不等baidu的所有元素，包括没有href的元素 | $(“a[href!=’baidu’]”). css(“background”,”red”)`
`$(“a[href^=’web’]”)` | 选择所有以web开头的元素 | `$(“a[href^=’web’]”). css(“background”,”red”)`
`$(“a[href$=’cn’]”)` | 选择所有以cn结尾的元素 |`$(“a[href$=’cn’]”). css(“background”,”red”)`
`$(“a[href*=’i’]”)` | 选择所有包含i这个字符的元素，可以是中英文 | `$(“a[href*=’i’]”). css(“background”,”red”)`
`$(“a[href][title=’我’]”)` | 选择所有符合指定属性规则的元素，都符合才会被选中。 | `$(“a[href][title=’我’]”). css(“background”,”red”)`

### 选择器汇总

-  **ID选择器**（js一般尽量用ID选择器，效率最高）：`$("#id").html();`
-  类选择器：`$(".className").text();`
-  标签选择器：`$('p').click();`
-  属性选择器：`$("li[id]")、 $("li[id='link']").fadeIn();`
-  层级选择器：`$("li .link").show();`
-  父子选择器：`$("ul > li")`
-  伪类选择器：`$("p:first") `
-  表单选择器：`$(":text") `

```javascript
*               $("*")              所有元素
#id             $("#lastname")      id="lastname" 的元素
.class          $(".intro")         所有 class="intro" 的元素
element         $("p")              所有 <p> 元素
.class.class    $(".intro.demo")    所有 class="intro" 且 class="demo" 的元素


:first  $("p:first")    第一个 <p> 元素
:last   $("p:last")     最后一个 <p> 元素
:even   $("tr:even")    所有偶数 <tr> 元素
:odd    $("tr:odd")     所有奇数 <tr> 元素

          
:eq(index)      $("ul li:eq(3)")    列表中的第四个元素（index 从 0 开始）
:gt(no)         $("ul li:gt(3)")    列出 index 大于 3 的元素 greater than
:lt(no)         $("ul li:lt(3)")    列出 index 小于 3 的元素 less than
:not(selector)  $("input:not(:empty)")  所有不为空的 input 元素
         
:header         $(":header")        所有标题元素 <h1> - <h6>
:animated       所有动画元素

            
:contains(text)     $(":contains('W3School')")  包含指定字符串的所有元素
:empty              $(":empty")                 无子（元素）节点的所有元素
:hidden             $("p:hidden")               所有隐藏的 <p> 元素
:visible            $("table:visible")          所有可见的表格
         
s1,s2,s3            $("th,td,.intro")            所有带有匹配选择的元素

            
[attribute]         $("[href]")         所有带有 href 属性的元素
[attribute=value]   $("[href='#']")     所有 href 属性的值等于 "#" 的元素
[attribute!=value]  $("[href!='#']")    所有 href 属性的值不等于 "#" 的元素
[attribute$=value]  $("[href$='.jpg']") 所有 href 属性的值包含以 ".jpg" 结尾的元素

           
:input      $(":input")         所有 <input> 元素
:text       $(":text")          所有 type="text" 的 <input> 元素
:password   $(":password")      所有 type="password" 的 <input> 元素
:radio      $(":radio")         所有 type="radio" 的 <input> 元素
:checkbox   $(":checkbox")      所有 type="checkbox" 的 <input> 元素
:submit     $(":submit")        所有 type="submit" 的 <input> 元素
:reset      $(":reset")         所有 type="reset" 的 <input> 元素
:button     $(":button")        所有 type="button" 的 <input> 元素
:image      $(":image")         所有 type="image" 的 <input> 元素
:file       $(":file")          所有 type="file" 的 <input> 元素

           
:enabled    $(":enabled")   所有激活的 input 元素
:disabled   $(":disabled")  所有禁用的 input 元素
:selected   $(":selected")  所有被选取的 input 元素
:checked    $(":checked")   所有被选中的 input 元素
```

### jQuery选择方法

获取父级元素

```javascript
    * $(selector).parent();     //获取直接父级
    * $(selector).parents('p'); //获取所有父级元素直到html   
```

获取子代和后代的元素

```javascript
    * $(selector).children();   //获取直接子元素
    * $(selector).find("span"); //获取所有的后代元素
    * find方法 可能用的多。
```

获取同级的元素

```javascript
    * $(selector).siblings()    //所有的兄弟节点
    * $(selector).next()        //下一个节点
    * $(selector).nextAll()     //后面的所有节点
    * $(selector).prev()        //前面一个的兄弟节点
    * $(selector).prevAll()     //前面的所有的兄弟节点
```

过滤方法

```javascript
    * $("div p").last();        //取最后一个元素
    * $("div p").first();       //取第一个元素
    * $("p").eq(1);             //去第n个元素
    * $("p").filter(".intro");  //过滤，选择所有p标签带有 .intro类，与$('p.intro')效果相同
    * $("p").not(".intro");     //去除，跟上面的filetr正好相反
```

---
## 4 Dom操作

### jQuery对象与DOM对象的互相转换

jQuery对象转换成DOM对象：

- 方式一：`$(“#btn”)[0]`
- 方式二：`$(“#btn”).get(0)`

DOM对象转换成jQuery对象：

```javascript
    $(document); //把DOM对象转成了jQuery对象
    var btn = document.getElementById(“bt n”);
    var jObj = $(btn); //把dom对象btn转换成jQuery对象
```

### 获取html的内容

- `$(selector).text()`：设置或返回所选元素的文本内容
- `$(selector).html()`：设置或返回所选元素的内容（包括 HTML 标记）
- `$(selector).val()`：设置或返回表单字段的值

获取和设置相同方法名，通过不同参数来确定是获取还是设置值，如果调用方法时没有传递参数，则表明时获取值。

```javascript
    $("#blin").text("你好Web"); //设置
    var txt = $("#blin").text(); //获取
```

使用html来创建dom的方式效率比较高于`document.createElement();`。

### 基本样式操作

基本样式操作方法：

```javascript
    $(selector).css("color","red")) 设置或返回匹配元素的样式属性。也可以通过对象传入多个样式，比如：css({})
    $(selector).width()             设置或返回匹配元素的宽度。    
    $(selector).height()            设置或返回匹配元素的高度。
               .innerHeight()       只能获取：内边距+内容的高度
               .innerWidth()
               .outerWidth()        获取：左右内边距+内容+左右边框  
               .outerHeight()
    $(selector).offset()            返回第一个匹配元素相对于文档的位置。比如：{ left:99, top: 22}
    $(selector).offsetParent()      返回最近的定位祖先元素。
    $(selector).position()          返回第一个匹配元素相对于父元素的位置。
    $(window).scrollLeft()          设置或返回匹配元素相对滚动条左侧的偏移。
    $(window).scrollTop()           设置或返回匹配元素相对滚动条顶部的偏移。可以通过$(selector).on("scroll",function(){});
```

样式类操作**尽量操作样式类，少直接操作css属性**，即先css预定义样式类，然后动态的修改元素的类：

```javascript
    $(selector).addClass('class');     向匹配的元素添加指定的类名。
    $(selector).removeClass('class');  从所有匹配的元素中删除全部或者指定的类。
    $(selector).toggleClass('class')   从匹配的元素中添加或删除一个类。
    $(selector).hasClass('class')      检查匹配的元素是否拥有指定的类。
```

注意：

- `height()`方法和`css(“height”)`的区别：返回值不同，`height()`方法返回的是数字类型(20)，`css(“height”)`返回的是字符串类型(20px)。
- `$(“div”).offset();`获取或设置坐标值，设置值后元素变成相对定位，传入参数必须包括：left和top属性。比如： `{left：100，top：150}`
- `$(“div”).position();`获取坐标值，适用于类似**子绝父相**定位，只能读取不能设置
- `$(“div”).scrollTop();`相对于滚动条顶部的偏移
- `$(“div”).scrolllLeft();`相对于滚动条左部的偏移

### 属性操作

```javascript
    $(selector).attr("id")      设置或返回匹配元素的属性和值
    $(selector).removeAttr()    从所有匹配的元素中移除指定的属性
    
    $(selector).prop("id")      获取在匹配的元素集中的第一个元素的属性值。
```

### 动态创建

```javascript
    $(selector).append(node)    在被选元素的结尾插入内容
    $(selector).append('<div></div>')

    $(selector).appendTo();     追加到参数指定的元素中

    $(selector).prepend()       在被选元素的开头插入内容
    $(selector).after()         在被选元素之后插入内容
    $(selector).before()        在被选元素之前插入内容
```

---
## 5  事件

### 简单事件绑定方法

- `click(hander)、click()` 绑定事件或者触发click事件
- `blur()` 失去焦点事件
- `hover(mousein, mouseleave)` 鼠标移入，移出
- `dbclick()` 双击
- `change` 改变，比如：文本框发送改变，下来列表发生改变等
- `focus()` 获得焦点
- `keyup(), keydown(), keypress()` 键盘和按钮事件
- `mouseout()` 当鼠标离开元素及它的子元素的时都会触发
- `mouseleave()` 当鼠标离开自己时才会触发，子元素不触发
- `mousedown()` 当鼠标指针移动到元素上方，并按下鼠标按键时
- `mouseover()` 当鼠标指针位于元素上方时

mouseover、mouseenter 事件的区别：

- `mouseover/mouseout`事件，鼠标经过的时候会触发多次，每遇到一个子元素就会触发一次。
- `mouseenter/mouseleave`事件，鼠标经过的时候只会触发一次


###  bind绑定方式

不推荐，1.7以后的jQuery版本被on方法取代，语法格式：`bind( eventType [, eventData ], handler )`

参数说明:

* 第一个参数：事件类型
* 第二个参数：传递给事件响应方法的数据对象(可以省略)，事件响应方法中获取数据方式：`e.data`
* 第三个参数：事件响应方法

```javascript
    $("p").bind("click", function(e){
        //事件响应方法
    });

    $("p").on('click',function(e){
        //事件响应方法
    })
```

### delegate绑定方式

推荐使用delegate，性能高，支持动态创建的元素。

 语法格式：`$(selector).delegate( selector, eventType, handler )`

语法说明：

- 第一个参数:selector， 子选择器
- 第二个参数：事件类型
- 第三个参数：事件响应方法

```javascript
        $(".parentBox").delegate("p", "click", function(){
            //为 .parentBox下面的所有的p标签绑定事件
        });

        $(".parentBox").on("click","p", function(){
            //为 .parentBox下面的所有的p标签绑定事件
        });
```

### one绑定方式(只绑定一次事件)

语法：`one( events [, data ], handler )`

```javascript
    $( "p" ).one( "click", function() {
      alert( $( this ).text() );
    });

    $("p").on("click",function(){
        $(this).off('click');//事件方法执行了一次后，就立即解绑事件
    })
```

###  on绑定方式()
 
jQuery 1.7版本后，jQuery用on统一了所有的事件处理的方法，一般建议使用这种方式

语法格式：`$(selector).on( events [, selector ] [, data ], handler )`

参数介绍：

* 第一个参数：events，事件名
* 第二个参数：selector，类似delegate
* 第三个参数: 传递给事件响应方法的参数
* 第四个参数：handler，事件处理方法

```javascript
    //绑定一个方法
    $( "#dataTable tbody tr" ).on( "click", function() {
      console.log( $( this ).text() );
    });

    //给子元素绑定事件
    $( "#dataTable tbody" ).on( "click", "tr", function() {
      console.log( $( this ).text() );
    });

    //绑定多个事件的方式
    $( "div.test" ).on({
      click: function() {
        $( this ).toggleClass( "active" );
      }, mouseenter: function() {
        $( this ).addClass( "inside" );
      }, mouseleave: function() {
        $( this ).removeClass( "inside" );
      }
    });
```

### 事件解绑

- unbind：解绑bind方式绑定的事件(在jQuery1.7以上被 on和off代替)
    * `$(selector).unbind();` 解绑所有的事件
    * `$(selector).unbind("click");`解绑指定的事件
- undelegate解绑delegate事件
    * `$( "p" ).undelegate();`//解绑所有的delegate事件
    * `$( "p" ).undelegate( "click" );`//解绑所有的click事件
- off解绑on方式绑定的事件
    * `$( "p" ).off();`
    * `$("P").off('click');`
    * `$( "p" ).off( "click", "**" );` 解绑所有的click事件，两个\*表示所有
    * `$( "body" ).off( "click", "p", foo);`

```javascript
    案例1：
    var foo = function() {
      // Code to handle some kind of event
    };
     
    // ... Now foo will be called when paragraphs are clicked ...
    $( "body" ).on( "click", "p", foo );
     
    // ... Foo will no longer be called.
    $( "body" ).off( "click", "p", foo );

    案例2：（了解）解绑命名空间的方式：
    var validate = function() {
      // Code to validate form entries
    };
     
    // Delegate events under the ".validator" namespace
    $( "form" ).on( "click.validator", "button", validate );
     
    $( "form" ).on( "keypress.validator", "input[type='text']", validate );
     
    // Remove event handlers in the ".validator" namespace
    $( "form" ).off( ".validator" );
```

### 触发事件

- 简单事件触发：`$(selector).click();`，触发 click事件
- trigger方法触发事件：`$( "#foo" ).trigger( "click" );`
- triggerHandler触发事件响应的方法，不触发浏览器行为：`$( "input" ).triggerHandler( "focus" );`

### 事件冒泡

在一个对象上触发某类事件（比如单击onclick事件），如果此对象定义了此事件的处理程序，那么此事件就会调用这个处理程序，如果没有定义此事件处理程序或者事件返回true，那么这个事件会向这个对象的父级对象传播，从里到外，直至它被处理（父级对象所有同类事件都将被激活），或者它到达了对象层次的最顶层，即document对象（有些浏览器是window）

### event对象

- `event.data` 传递的额外事件响应方法的额外参数
- `event.currentTarget === this` 在事件响应方法中等同于this，当前Dom对象
- **`event.target`** 事件触发源，不一定`===this`
- **`event.pageX`** The mouse position relative to the left edge of the document
- **`event.pageY`** The mouse position relative to the top edge of the document
- **`event.stopPropagation()`** 阻止事件冒泡
- **`e.preventDefault();`** 阻止默认行为
- `event.type` 事件类型：click，dbclick...
- `event.which` 鼠标的按键类型：左1 中2 右3
- `keydown` 按键，比如：`a,b,c`
- `event.keyCode` code的c是大写

----
## 6 动画

动画相关方法：`show、hide、fadeIn、fadeOut、fadeTo、slideDown、slideUp、slideToggle、animate`

### 隐藏显示

```javascript
$(selector).show(speed,callback);
$(selector).hide(1000);
$(selector).toggle("slow");
```    

三个方法的语法都一致，参数可以有两个，第一个是动画的速度，第二个是动画执行完成后的回调函数。第一个参数是：可以是单词或者毫秒数

### 淡入淡出

```javascript
    $(selector).fadeIn(speed, callback)
    $(selector).fadeOut(1000)
    $(selector).fadeToggle('fast',function(){})
    $(selector).fadeTo(.5); //淡入到0透明，1不透明
```

### 滑动

```javascript
$(selector).slideDown(speed,callback);
$(selector).slideUp(speed,callback);
$(selector).slideToggle(speed,callback);
```

### 自定义动画

函数：`$(selector).animate({params},speed,callback);`

- animate动画：不支持背景的动画效果
- animate动画支持的属性列表


```javascript
    $("button").click(function(){
      $("div").animate({
        left:'250px',
        opacity:'0.5',
        height:'150px',
        width:'150px'
      },2000);
    }).animate({},1000); 
```

### 结束动画

```javascript
$(selector).stop()
$(selector).stop(stopAll,goToEnd);
```

- clearQueue：如果设置成true，则清空队列。可以立即结束动画。
- gotoEnd：让当前正在执行的动画立即完成，并且重设show和hide的原始样式，调用回调函数等。

----
## 7 jQuery补充

### each和map函数

```javascript

// 全局的each
$.each(array, function(index, object){})
// 普通jQuery对象的each方法，参数的顺序是一致的。
$("li").each(function(index, element){} )

//全局的map函数，返回变换后的数组，注意参数的顺序是反的
$.map(arry,function(object,index){})
$("li").map(function(index, element){})


//示例：

        var newArr = $.map($("li"), function(i, e) {
            return $(e).text() + i;//每一项返回的结果组成新数组
        });

        var newArr = $("li").map(function(elem, index){
            console.log("elem:" + elem);
            console.log("index:" + index);
            retrun index;
        });
```

### `$()`的几种常用用法

1. `$(“#id”)` 选择某个元素，返回值为jQuery对象
2. `$(this)` 将DOM对象转换成jQuery对象
3. `$(“<div></div>”)` 创建元素，返回值为jQuery对象
4. `$(function(){})` 入口函数的简写方式

### 链式编程

`$("div").html()` 后面就不能继续链式了？为什么？函数返回值不再是之前的jQuery对象了，此时需要调用`end()`方法来重新回到开头的jQuery对象

### 隐式迭代

* 默认情况下，会自动迭代执行jQuery选择出来所有dom元素的操作。
* 如果获取的是多元素的值，默认返回的是第一个元素的值。

$("div").html() 获取内容的时候，只返回匹配到的第一个元素的数据

### 数据缓存

可用通过jQuery提供的data方法缓存数据

- `$(“div”).data(key);`获取数据
- `$(“div”).data(key,"value");`添加数据

```html
<div data-role="page" data-last-value="43" data-hidden="true" data-options='{"name":"John"}'>
注意HTML属性不区分大小写
</div>

$( "div" ).data("lastValue" ) === 43，变换过程：lastValue -> data-last-value
```

`data()`跟`attr()`方法的区别：

1. 获取数据的时候，attr方法需要传入参数，data方法可以不传入参数，这时候获取到的是一个js对象，即使没有任何data属性。
2. 获取到的数据类型不同，attr方法获取到的数据类型是字符串(String)，data方法获取到的是相应的类型。
3. data方法获取到数据之后，我们使用一个对象来接收它，那么就可以直接操作(设置值或获取值)这个对象，而attr方法不可以。并且通过这种方式，要比`data(key,value)`的方式更高效。
4. `data-attribute`属性会在页面初始化的时候放到jQuery对象，被缓存起来，而attr方法不会。


### noConflict

 两个全局的对象：`$、jQuery`

```javascript
    //第一步，定义一个 $ 对象
    var $ = { name : "normal" };
    //第二步，引入jQuery库，此时：$ === jQuery
    <script src="jQueryDemos/jquery-1.11.3.min.js"></script>
    //第三步：让jQuery释放 $, 让$回归到jQuery之前的对象定义上去。
    var laoma_jQ = $.noConflict();
    //第四步：$现在是我们定义的对象，laoma_jQ是jQuery全局对象
```

----
## 8 插件

- Validate：表达校验库
- jQueryUI：UI库
- lazyload：懒加载
- color：支持对背景色做动画

怎么写插件：

```javascript
(function ($) {
    $.fn.lxCfbg = function (options) {
        // 合并参数
        var opts = $.extend({
            "color": "#000",
            "font-size": "16px",
            "background-color": "#fff"
        }, options);
        $this.css({
            "color": opts.color,
            "font-size": opts["font-size"],
            "background-color": opts["background-color"]
        });
        return $this;
    };
})(jQuery);
```

---
## 9 开发中如何使用jQuery

- 参考笔记
- 参考做过的示例
- 查看文档：chm、MDN、官方文档
- 书籍：《jQuery权威指南》、《锋利的jQuery》