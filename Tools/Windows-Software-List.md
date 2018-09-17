# 软件列表

---
## 链接

- [Windows 下有什么软件能够极大地提高工作效率？](https://www.zhihu.com/question/22919326)
- [Windows 10 上有什么值得推荐的软件？](https://www.zhihu.com/question/36847530)
- [高效 Tools 系列](https://jeffjade.com/tags/Toss/)
- [少数派](https://sspai.com/topics)


---
##  IDE

- IDEA
- CLion
- visual studio
- Source Insight
- SciTools
- AndroidStudio
- Sublime Text3
- Atom
- Cmake
- vscode


---
## 数据库

- [Android逆向助手](http://www.dayanzai.me/android-hack.html)
- [ClassyShark](https://github.com/google/android-classyshark/releases)
- SqliteExpert
- MySql
- Oracle


---
## 类Unix

- CygMin
- MinGW
- [GOW](https://github.com/bmatzelle/gow)
- Chocolatey
 - `ChocolateyInstall` 配置安装环境
 - `ChocolateyToolsRoot` 和 `ChocolateyToolsLocation`  配置软件安装环境
- [babun](http://babun.github.io/)
- [cmder](http://cmder.net/)
 - `Cmder.exe /REGISTER ALL`
 - Environment：`set LANG=zh_CN.UTF-8`
 - [Win下必备神器之Cmder](https://jeffjade.com/2016/01/13/2016-01-13-windows-software-cmder/)
- [xshell-cn](http://www.xshellcn.com/xshell.html)
- Windows10-bash子系统：详细配置参考[这里](http://www.jianshu.com/p/bc38ed12da1d)
- LLVM (要求安装 visual studio)


---
##  网络

- Fiddler
- Charles
- Wireshark
- netassist
- Cisco Packet Tracer


---
## 开发

- WinHex：16进制编辑器
- NodeJs + NVM
- Genymotion：模拟器


---
## 压缩

- 7zip
- [bandizip](http://www.bandisoft.com/bandizip/)

---
##  UI设计

- [马克鳗](http://www.getmarkman.com/)
- Photoshop:[adobe系列](http://adobe.v404.cn/adobe/)
- Sketeh

---
##  高效工具

- Everything
- Listary
- WOX
- LICEcap：gif制图
- Snipaste：截图+贴图
- [picpick](http://ngwin.com/picpick/readme)：截图取色
- ShareX
- AutoHotkey
- Seer 或 QuickLook

---
##  画图

- Astah
- StarUML
- xmind

---
##  图片压缩

- [iSparta](http://isparta.github.io/)
- [智图](http://zhitu.isux.us/index.php/preview/download)
- [PngQuant](https://pngquant.org/)
- Caesium boxed
- [magicexif](http://www.magicexif.com/)
- [mediainfo-gui](s://sourceforge.net/projects/mediainfo/files/binary/mediainfo-gui/0.7.58/)

---
##  常用软件

- ccleaner
- 为知笔记
- EasyBCD
- EasyEFI
- 分区助手
- 完美解码
- UltralSO 刻录
- MPV.io
- 鲁大师
- Total Commander
- video-converter
- eagleget下载器
- 百度云盘

---
## Windows

- 微信
- 网易云
- 123看图器
- Photo Editor
- Ditto 剪切板增强
- OneDrive
- 微软必应词典
- Cortana搜索预览
- Win + G 录屏
- 云剪贴板：`“设置”→“系统”→“剪贴板”`
- Windows ink 工作区
- 强大的计算器
- OneNote

### 1 Sublime


安装包管理工具：

```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

解决windows下文件夹名为方框：

1. 打开 Preference->Settings
2. 最外层 json 加入 `"dpi_scale": 1.0`
3. 重启

常用插件：

- Localization
- CTag
- Git
- Side Bar
- ConvertToUTF8
- SublimeLinter
- Terminal

### 2 SciTools Unstand

- [推荐：Mac下高效静态代码分析神器Unstand详解](http://mp.weixin.qq.com/s?__biz=MzA5MTUzODAzNw==&mid=2650744718&idx=1&sn=e739b6d211e913721ad0f09ce65d2fa1&scene=2&srcid=0502oierWP6qyZopRaXWhvif&from=timeline&isappinstalled=0#wechat_redirect)

### 3 vscode

markdown：

```
background: #272822;
color: white;

h1,h2,h3,h4,h5,h6,table,tr,td,th,strong{
color: white;
}
```

encodeing：
```
files.autoGuessEncoding = true
```
