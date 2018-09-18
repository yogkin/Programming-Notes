## gradle

```
sudo add-apt-repository ppa:cwchien/gradle
sudo apt-get update
sudo apt-get install gradle

export GRADLE_HOME=/opt/gradle-4.2.1
export PATH=$GRADLE_HOME/bin:$PATH
```

## SDK

配置SDK环境变量

```
下载
wget http://dl.google.com/android/android-sdk_r24.2-linux.tgz
tar -xvf android-sdk_r24.2-linux.tgz
cd android-sdk-linux/tools
执行 .android 打开图形化界面更新AndroidSDK

配置环境变量
export ANDROID_HOME=$HOME/android-sdk-linux
export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools"
```

## AndroidStudio

1. 官网下载linux版本的AndroidStudio
2. 解压到文件夹下(放在HOME目录下即可)
3. 导航至` android-studio/bin/` 目录，并执行 `. studio.sh`。启动AndroidStudio，之后AndroidStudio设置向导将指导您完成余下的设置，包括下载开发所需的AndroidSDK组件

如果运行AndroidStudio构建项目是遇到问题
```
在Android Studio文件夹下执行下面命令
sudo chmod 777 * -R
```

如果运行的是64位版本Ubuntu，则需要使用以下命令安装一些32位库
```
# 执行下面命令
sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0 lib32stdc++6`
apt-get install libbz2-1.0
apt-get install lib32z1 lib32ncurses5 lib32stdc++6
# 或者上门运行错误的话执行下面命令
sudo apt-get install -y libc6-i386 lib32stdc++6 lib32gcc1 lib32ncurses5 lib32z1
# 或者上门运行错误的话执行下面命令
# adb
sudo apt-get install libc6:i386 libstdc++6:i386
# aapt
sudo apt-get install zlib1g:i386

```

## shadowsoks

```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```

## 安装字体

```
apt-get install msttcorefonts

字体将会被安装到 /usr/share/fonts/truetype/msttcorefonts

　　分别是：
　　Andale Mono, Arial, Comic Sans MS, Courier New, Georgia
　　Impact, Times New Roman, Trebuchet MS, Verdana, Webdings
```


## 设置环境变量

按变量的生存周期来划分，Linux变量可分为两类：

1. 永久的：需要修改配置文件，变量永久生效。
2. 临时的：使用export命令声明即可，变量在关闭shell时失效。


- 1 在`/etc/profile`文件中添加变量【对所有用户生效(永久的)】
```
vim /etc/profile
添加环境变量：
    export PATH=$PATH:你的路径
退出后：
source /etc/profile
```

- 2 在用户目录下编辑`.bashrc`文件【对单一用户生效(永久的)】
```
vim ~/.bashrc #编辑配置文件
添加环境变量：
       export YOURPATH=xxx/xxx
       export PATH=$PATH:$YOURPATH
退出后
source ~/.bashrc #更新环境变量
```

- 3 直接运行export命令定义变量【只对当前shell(BASH)有效(临时的)】
```
export PATH=$PATH:你的路径

该变量只在当前的shell(BASH)或其子shell(BASH)下是有效的，shell关闭了，变量也就失效了，再打开新shell时就没有这个变量，需要使用的话还需要重新定义
```

## 参考


- [shadowsocks-qt5](https://github.com/shadowsocks/shadowsocks-qt5)
- [ubuntu-for-Android](https://github.com/gaoneng102/ubuntu-for-Android)