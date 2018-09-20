# FFmeng资料

- [ffmpeg](https://ffmpeg.org/)

---
## 1 编译

- [FFmpeg官方编译指引](https://trac.ffmpeg.org/wiki/CompilationGuide)
- [ffmpeg-android-java](https://github.com/WritingMinds/ffmpeg-android-java)：Android端命令行
- [雷霄骅——最简单的基于FFmpeg的移动端例子：Android HelloWorld](http://blog.csdn.net/leixiaohua1020/article/details/47008825)
- [编译可供Android使用的FFmpeg库](http://blog.crazyman.top/2015/05/06/%E7%BC%96%E8%AF%91ffmpeg/):编译出单个库，ndkbuild方式
- [BlogDemo](https://github.com/burgessjp/BlogDemo)：CMake方式
- [Android最简单的基于FFmpeg的例子](http://www.ihubin.com/archives/)
- [FFmpeg4Android](https://github.com/mabeijianxi/FFmpeg4Android)：这是一个编译 Android 下可用的 FFmpeg 的项目，内含代码示例。包含libx264全平台编译脚本、libfdk-aac全平台编译脚本。

### 配置

- configure详细指令说明可以从`./configure --help`命令获取
- configure文件中描述了各个模块之间的依赖关系等

```
configure文件中的下面内容：
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'

替换为：
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

---
## 库

- [VideoCompression](https://github.com/RudreshJR/VideoCompression)，Android Library for VideoCompressionLibrary for VideoCompression

---
## 学习

- [FFMPEG视音频编解码零基础学习方法](http://blog.csdn.net/leixiaohua1020/article/details/15811977)
- [WeiXinRecordedDemo](https://github.com/Zhaoss/WeiXinRecordedDemo)