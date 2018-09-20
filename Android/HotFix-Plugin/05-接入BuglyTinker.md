# 接入Bugly Tinker

---
## 1 热修复选择

- Bugly+tinker：接入相对简单，然后加上Bugly免费而强大的后台管理。
- robust：接入侵入性大，后台需要自己开发。
- Sophix：傻瓜式接入，但是阿里云的后台是收费的。
- [Android 热更新服务平台](http://tinkerpatch.com/)+tinker：接入相对简单，但是tinkerpatch是收费的。

最终选择：bugly+tinker

---
## 2 如何集成

1. Bugly申请账号，创建应用，获取APP ID
2. 接入Tinker Support(Tinker Support已经包括了Crash统计)
3. 根据文档配置gradle脚本

### 关于上传补丁

1. 上传补丁前需要对应的APK上报联网，然后上传补丁时会判断是否有该补丁对应的apk，如果没有则不允许上传补丁
2. 上传补丁的路径，是：`build/outputs/patch/patch_signed_7zip.apk`，不要搞错了

### 关于多渠道打包于加固

Tinker的补丁是基于DexDiff算法生成的，所以如果使用gradle的多渠道打包方法会存在问题，即gradle的多渠道打包生成的各个渠道
的apk的dex是不一样的，那样生成补丁时就会针对每个渠道包生成一个补丁，这样下发补丁就会非常麻烦，bugly也不支持根据渠道发布补丁。

所以不能使用gradle的多渠道打包工具，那么可以使用第三方的多渠道打包工具：

- [VasDolly](https://github.com/Tencent)
- [walle](https://link.jianshu.com/?t=https://github.com/Meituan-Dianping/walle)
- [packer-ng-plugin ](https://github.com/mcxiaoke/packer-ng-plugin)

这些工具打包将渠道信息写在apk文件的zip comment中，不会造成多渠道包的dex差异。

如果要使用加固，则tinker的gradle配置脚本中isProtectedApp要设置为true，然后多渠道打包工具要选择VasDolly。

### 具体步骤

1. 先打基础包(这是用于打 tinker 补丁的基础包)
1. 然后进行加固
1. 加固后手动进行 zipalign，命令行：`zipalign -v 4 old.api new.apk`，(AndroidStudio 打出的正式包是已经 zipalign 的，但360加固后需要重新 zipalign，其他加固平台未知)
1. zipalign 后再进行签名(这也可以认为是加渠道信息的基础包)
1. 添加渠道信息
1. 出现 Bug，根据步骤1生成的 apk 打补丁，然后发布补丁


---
## 引用

- [Tinker热修复集成总结](https://www.jianshu.com/p/194c9d89b227)
- [VasDolly](https://github.com/Tencent/VasDolly)
- [tinker](https://github.com/Tencent/tinker/wiki)
- [Bugly](https://bugly.qq.com/v2/index)

