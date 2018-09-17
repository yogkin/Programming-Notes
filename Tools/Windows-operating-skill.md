## 1  Windows常用的快捷键

```
*   WIN+D：显示桌面，再按一次还原桌面；
*   WIN+R：打开运行，输入命令可以执行相应操作
*   WIN+E：打开资源管理器；
*   CTRL+ALT+Delete：程序不响应时用这一招结束不响应的程序，xp下用得比较多；
*   WIN+L：锁屏；
*   WIN+Tab：切换程序；
*   ALT+SHIFT+TAB：切换程序；
*   CTRL+W：关闭资源管理器；
*   CTRL+Home：跳转到文件最开头，直接按Home跳转到行头；
*   CTRL+End：跳转到文件尾部，直接按End跳转到行尾；
*   CTRL+PageUP/PageDown：切换文档TAB
*   ALT+双击：查看文件属性；
*   WIN+数字键：启动任务栏上的程序；
*   在桌面或者任何文件夹下，SHIFT+鼠标右键，可以在当前路径下打开DOS命令窗口；
*   在桌面或者任何文件夹下，CTRL+鼠标左键，拖动文件、文件夹都可以立马生成文件对应的副本；
*   新建只有扩展名的文件的方法：”.suffix.”，比如创建.gitignore，正常情况下windows是不允许创建的，但在扩展名后面加点，即.gitignore.就可以正常创建了；
*   CTRL+SHIFT+ESC：打开进程管理器；
*   WIN+左箭头：当前窗口缩放为屏幕的一半，靠屏幕左侧显示；
*   WIN+右箭头：当前窗口缩放为屏幕的一半，靠屏幕右侧显示；
*   WIN+上箭头：最大化当前窗口；
*   WIN+下箭头：还原和最小化当前窗口；
*   在桌面上，右键任何一个程序，鼠标定位到快捷键一栏，为该应用设置启动快捷键，然后你就可以通过这个这个快捷键来启动该程序啦；
*   WIN+R，输入“msconfig”，弹出系统设置界面，可设置禁止、允许进程开机自启动；
*   WIN+Home：最小化所有窗口，除了当前激活窗口；
*   WIN+M：最小化所有窗口；
*   WIN+SHIFT+M：还原最小化窗口到桌面上；
*   WIN+P：选择一个演示文稿显示模式；
*   WIN+Pause：显示系统属性对话框；
*   WIN+F：打开windows帮助中心；
*   WIN+T：切换任务栏上的程序；
*   WIN+ALT+数字：让位于任务栏指定位置（按下的数字作为序号）的程序，显示跳转清单；
```


---
## 2 清理

### C 盘清理

- 依次进入Win10「设置」→「系统」→「存储」，然后开启「储存感知功能」，就可以让Windows自动清理临时文件。
- 定期清理C盘的垃圾文件，右键C盘选择磁盘清理选项。
- 以管理员身份运行：`powercfg hibernate size 40` ，压缩睡眠文件
- 清理软件
  - 软媒清理大师
  - cclearn
  - Dism ++
  - SpaceSniffer(查看磁盘占用)

### 注册表

清理右键：

- IE右键：`HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\MenuExt`
- 右键：`HKEY_CLASSES_ROOT\Directory\Background\shellex\ContextMenuHandlers`


----
## 3 Windows与Unix换行符号转换

由于两个系统的换行符不一样，有时候在Windows中的配置的类Unix运行shell脚本时就会出现问题。比如语法错误。

- Windows的换行是`回车加换行`
- Unix的换行是`回车`

使用notepad++可以解决问题。

- 使用notepad++显示所有符号：操作：`视图->显示符号->显示所有符号`
- 使用notepad++转换文档格式：操作：`编辑->档案格式转换->转换为UNIX格式`


---
## 4 Windows 10 版本历史

参考[Windows 10版本历史](https://zh.wikipedia.org/wiki/Windows_10%E7%89%88%E6%9C%AC%E5%8E%86%E5%8F%B2)

- 1607
- 1709
- 1803

---
## 5 Windows 命令行


### 1 获取帮助

使用`help 命令名`

### 2 命令

- **cd**：进入某个目录
- **tree**：以树形格式罗列文件
- **type**：显示一个文本的内容
- **cls**：清屏
- **findstr "关键字" 文件path**：查找文本
- `netstat -ano|findstr "8080"`查看端口占用

### 3 操作符

- `>`导出命令 比如`dir > D:\dir.txt`,就会输出到dir.txt中，而不是面板中
- `.`表示当前文件夹
- `..`表示上一级目录
- `../..`上一级目录的上一级目录
- `\`表示根目录

### 4 实用技巧

**当前目录创建a.txt文件**

    cd .>a.txt

**过滤字符**

    findstr "你要搜索的内容"  file.txt
    Select-String -path file.txt -Pattern  "你要搜索的内容"(powershell命令)


---
## 引用

- [Windows 有哪些你相见恨晚的奇技淫巧？](http://www.zhihu.com/question/27721113)
