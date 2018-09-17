# AndroidStudio

## 快捷键

功能|快捷键(名称)
---|---
显示方法调用层级|Ctrl + Alt + H
在文件中使用位置|Ctrl + F7
当移行上/下移一行|Alt + Shift + 上/下
不离开当前文件查看方法实现|Ctrl + Shift + I
添加书签/查看书签| F11 / Shift +F11
删除行，删除并复制行，复制行并粘贴|ctrl+y，ctrl+x, ctrl+d
关闭或者恢复其他窗口|ctrl+shift+f12
查看方法需要参数|ctrl+p
移动 / 重命名 | F6 / Shift + F6
多行编辑 | Alt + 摁住鼠标
查看文件结构| Ctrl + F12
智能提示 | Ctrl + Shift + Space
智能选择 扩大选择区域/减少选择区域 | Ctrl+W/Ctrl+shift + W
选中后可以使用 Extra 快捷键重构为变量、方法等|ctrl+ alt+ v：变量<br/>ctrl+ alt+ c：常量<br/>ctrl+ alt+ f：域值<br/>ctrl+ alt+ p：参数<br/>ctrl+ alt+ m：方法
大小写切换|Ctrl +Shift + U
打开类文件|Ctrl + N
打开file | Ctrl + Shift + N
最近打开文件窗口 | Ctrl + E
最近编辑过的文件窗口 | Ctrl+Shift + E
最后编辑的位置，可一直返回|ctrl+shift+backspace
被使用的地方（用于一个类、变量、方法）|Alt + F7
被使用的地方（用于一个类、变量、方法），显示在弹窗中|Ctrl + Alt + F7
抽象方法调到实现方法中| Ctrl + Alt + B
跳到父类 | Ctrl +U
高亮文件中所有同名单词 | Ctrl + Shfit + F7
智能方法上下选择 | Alt + 上/下
快速代码模板 | Ctrl + J
整个方法上下移动排序 | Ctrl + Shift + 上/下
打开Action | Ctrl + Shift + A
全局代码查找 | Ctrl + Shift + F
快捷的选择和打开文件目录|Ctrl + Alt + F12
ctrl+table | 实现各个界面切换
alt + 左/右|  操作历史
ctrl+./+/- | 代码折叠
ctrl+alt+shift+T|打开重构窗口(可以重构java代码，也可以重构xml布局等)
shift + F4|在新的窗口编辑文件


## 设置选项

1. **打开文件位置**：
右键文件--show in explorer
2. **检查升级**：
Settings  --> Updates
 *   **Stable Channel** ： 正式版本通道，只会获取最新的正式版本。
 *   **Beta Channel** ： 测试版本通道，只会获取最新的测试版本。
 *   **Dev Channel** ： 开发发布通道，只会获取最新的开发版本。
 *   **Canary Channel** ： 预览发布通道，只会获取最新的预览版本。
3. **自动导包**：
Editor--> General --> AutoImport
4. **代码字体**：
Editor--> Color&Font-->  Font
5. **代码颜色**：
比如java：
Editor--> Color&Font-->  Java
6. **生成文件自动签名**：
Editor--> File And Code Templates --> Includes
7. **显示文档注释**：
Editor--> General --> Other --> Show Quick documentation...
8. **智能提示，是否区分字母大小写**：
Editor-->General-->Code Completion
9. **显示行号**:
Editor-->General-->Appearance-->Show Line Number
10. **代码前缀**：
Editor—> Code Style—>
11. **关闭项目后，再次打开，不直接进入上次的项目**：
Appearance&Behavior-->System Setting -->Reopne last project on startup
12. **编辑快速生成代码提示**
Editor-->Live Templates
13. **驼峰选择**
Editor-General-->Smart Keys-->Use CameHumps Words
14. **Log颜色**：
比如java：
Editor--> Color&Font-->  Android Log


## 相关工具

### 反编译

- **[android-classyshark——apk分析利器](https://github.com/google/android-classyshark)**
- [ClassyShark——apk分析利器](http://w4lle.github.io/2016/02/15/ClassyShark%E2%80%94%E2%80%94%E5%88%86%E6%9E%90apk%E5%88%A9%E5%99%A8/)
- [Google官方出品的将Dalvik字节码转换为等价的Java字节码的工具](https://github.com/google/enjarify)
- [FontZip字体资源文件压缩神器](https://github.com/forJrking/FontZip)


### Keystore 密码恢复

- [Android Keystore Password Recovery](http://maxcamillo.github.io/android-keystore-password-recover/)

### AndroidStudio 3.0 DDMS

3.0 的 DDMS 已经被废弃，如果有需要，可以在 SDK 中打开，`android-sdk/tools/monitor.bat`








