# BOM

BOM包括五大对象

1. Window
2. Navigator：Navigator包含有关客户机浏览器的信息
3. Screen
4. History：浏览历史
5. Location：Location包含有关当前 URL 的信息

---
## Window对象

Window 对象表示浏览器中打开的窗口，如果文档包含框架（frame 或 iframe 标签），浏览器会为 HTML 文档创建一个 window 对象，并为每个框架创建一个额外的 window 对象。

- `window.frames` 返回窗口中所有命名的框架
- parent是父窗口（如果窗口是顶级窗口，那么`parent==self==top`），top是最顶级父窗口，self是当前窗口（等价window）
- opener属性表示用open方法打开当前窗口的那个窗口

### Window常用方法


- `open()/close()`：打开/关闭窗口
- `showModalDialog()`：显示模态框
- `window.alert(String)、window.confirm(String)、window.prompt(String)`：展示警告、确认、输入对话框，confirm返回布尔值，prompt返回输入的内容
- `setInterval(code,millisec[,"lang"])`：按照指定的周期（以毫秒计）来调用函数或计算表达式
- `clearInerval()`：移除setInterval设置的函数调用任务
- `setTimeout(code,millisec)`：在指定的毫秒数后调用函数或计算表达式


---
## BOM History

History 对象包含用户（在浏览器窗口中）访问过的 URL

- `back()`  加载 history 列表中的前一个 URL
- `forward()`  加载 history 列表中的下一个 URL
- `go()`  加载 history 列表中的某个具体页面，可以是整数或负数


---
## BOM Location

Location 对象包含有关当前 URL 的信息，通过href属性完成超链接效果

```
window.location.href = "xxx.html"
```


---
## JavaScript常用事件

- 鼠标移动：`mousemove/mouseout/mouseover`
- 点击事件：`click/dblclick/mousedown/mouseup`
- 加载与卸载事件：`load/unload`
- 聚焦与离焦事件：`focus/blur`
- 键盘事件：`keydown/keyup/keypress`
- 提交与重置事件：`submit/reset`
- 选择与改变事件：`select/change`


```html
//鼠标移动
<input type="text" id="info" onmouseover="mouseovertest();"onmouseout="mouseouttest();"/>

//加载与卸载事件
<body onload=“loadform()” onbeforeunload=“unloadform()”>
      <a href=“http://www.google.cn”>谷歌</a>
</body>

//键盘事件
<script language="JavaScript">
function submitform(e){
    if(e.keyCode){
        if (e.keyCode == 13) {document.forms(0).submit();}  
    }else{
        if (e.which == 13) {document.forms(0).submit();}  
    }
           }
</script>

<input type="text" name="username" onkeypress="submitform(event);"/>

//提交事件
<form action="delete" onsubmit="return isDelete();">
姓名：小明  年龄：23  学校：清华大学
<input type="submit" value="删除">
</form>
```

