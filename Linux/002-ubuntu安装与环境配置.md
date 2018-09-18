## 1 安装 ubuntu（双系统）

预备设置：windiws关闭快速启动，bios关闭安全启动。

1. U盘启动
2. uefi警告时选择`后退`
3. 分区为：
 - `/Boot`(引导分区) ，1000M ，主分区
 - `/`(根分区) ，30760M ，这里不要设置的太小，毕竟可能安装许多软件
 - 交换分区，8000M。
 - `/home`(用户目录), 剩余所有容量
4. 安装完毕
5. 进入windows10，使用easyBCD添加引导项：`grub2-自动选择`


---
## 2 ubuntu环境搭建与相关配置

### install unity tools

```
        sudo add-apt-repository ppa:freyja-dev/unity-tweak-tool-daily
        sudo apt-get update
        sudo apt-get install unity-tweak-tool
```

### install aptitude

aptitude工具是基于apt的一款安装工具，优点是可以自动解决安装和卸载时候的依赖关系。

```
        sudo apt install aptitude
```

### 安装工具

```
        sudo aptitude install shutter//安装截屏软件shutter
        sudo apt install docky//docky工具栏：使用 apt/apt-get 可能需要重启才能生效，用aptitude安装后可以直接使用
```

### 安装主题

```
        //主题
        sudo add-apt-repository ppa:noobslab/themes
        sudo apt-get update
        sudo apt-get install flatabulous-theme
        //配套图标
        sudo add-apt-repository ppa:noobslab/icons
        sudo apt-get update
        sudo apt-get install ultra-flat-icons
```

### install jdk

```
        //首先安装软件包管理工具
        sudo apt-get install python-software-properties
        sudo apt-get install software-properties-common
        //Oracle
        sudo add-apt-repository ppa:webupd8team/java
        sudo apt-get update
        sudo apt-get install oracle-java8-installer
        //open jdk
        sudo apt-get update
        sudo apt-get install openjdk-8-jdk
```

### wiznote

```
        sudo add-apt-repository ppa:wiznote-team //添加官方源
        sudo apt-get update //更新源 
        sudo apt-get install wiznote //安装为知笔记
```

### oh-my-zsh

```
      sudo apt-get install zsh
```

### vim

```
sudo apt-get install vim-gtk
vim /etc/vim/vimrc #编辑配置文件，加入以下配置

    set nu
    set tabstop
    set cursorline
    set ruler
```

### dos2unix

```
    sudo apt-get install dos2unix
```

### Tomcat

从官网下载tomcat，然后解压即可，远程连接时要开放对应的端口。

```shell
# 创建tomcat存放目录，比如 /usr/local 目录下
cd /usr/local
mkdir tomcat

# 下载tomcat
wget http://mirrors.shu.edu.cn/apache/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.tar.gz

# 解压
tar -xvf apache-tomcat-8.5.31.tar.gz

# 启动tomcat(记得开放8080端口)
./startup.sh
./shutdown.sh
```


### MySQL

```
apt-get update
apt-get install mysql-server mysql-client
测试是否安装成功：netstat -tap | grep mysql

启动MySQL服务：service mysql start
停止MySQL服务：service mysql stop
服务状态：service mysql status
修改 MySQL 的管理员密码：mysqladmin -u root password newpassword

正常情况下，mysql占用的3306端口只是在IP 127.0.0.1上监听，拒绝了其他IP的访问（通过netstat可以查看到）取消本地监需要修改 my.cnf 文件：

  1 vim /etc/mysql/my.cnf，把 bind-address = 127.0.0.1注释掉，如果配置文件中没有bind-address = 127.0.0.1，则添加下面内容：
       [mysqld]
       bind-address = 0.0.0.0
  2 重启MySQL服务器
  3 重新登录 mysql -uroot -p
  4 在mysql命令行中运行下面两个命令
  grant all privileges on *.* to 'root'@'%' identified by '远程登录的密码';
    flush privileges;
  5 检查MySQL服务器占用端口 netstat -nlt|grep 3306

数据库存放目录： /var/lib/mysql/
相关配置文件存放目录：/usr/share/mysql
相关命令存放目录：/usr/bin(mysqladmin mysqldump等命令
启动脚步存放目录：/etc/rc.d/init.d/
```

sql授权说明

语法：`grant 权限1,权限2, ... 权限n on 数据库名称.表名称 to 用户名@用户地址 identified by '连接口令';`

- `权限1，权限2，... 权限n` 代表 `select、insert、update、delete、create、drop、index、alter、grant、references、reload、shutdown、process、file` 等14个权限。
- 当`权限1，权限2，... 权限n` 被 `all privileges` 或者 `all` 代替时，表示赋予用户全部权限。
- 当 `数据库名称.表名称` 被 `*.*` 代替时，表示赋予用户操作服务器上所有数据库所有表的权限。
- 用户地址可以是localhost，也可以是IP地址、机器名和域名。也可以用 `'%'` 表示从任何地址连接。
- '连接口令'，远程连接时使用的密码，不能为空，否则创建失败。

>privileges 即特权的意思。

### Nginx


```shell
# 先下载安装  nginx 和 nginx-rtmp 编译依赖工具
sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev

# 下载 nginx 和 nginx-rtmp源码（wget是一个从网络上自动下载文件的自由工具）
wget http://nginx.org/download/nginx-1.7.5.tar.gz

# 解压 nginx
tar -zxvf nginx-1.7.5.tar.gz

# 切换到 nginx-目录
cd nginx-1.7.5

# 编译安装nginx
make
sudo make install

# 安装nginx init脚本
sudo wget https://raw.github.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx
sudo chmod +x /etc/init.d/nginx

# 开启启动nginx
sudo update-rc.d nginx defaults

# 启动和停止nginx 服务，生成配置文件
sudo service nginx start
sudo service nginx stop
sudo service nginx restart
```


### 开放端口号

```
查看哪些端口被打开  netstat -anp
关闭端口号：iptables -I INPUT -p tcp --drop 端口号 -j DROP
　         iptables -I OUTPUT -p tcp --dport 端口号 -j DROP

打开端口号：iptables -I INPUT -ptcp --dport  端口号 -j ACCEPT
将修改永久保存到防火墙中：iptables-save

关闭/打开防火墙（需要重启系统）
开启： chkconfig iptables on
关闭： chkconfig iptables off
```

---
## 引用

- [Ubuntu 16.04安装Java JDK](http://topspeedsnail.com/ubuntu16-install-java-jdk/)
- [Windows10+Ubuntu双系统安装[多图]](http://www.jianshu.com/p/2eebd6ad284d)
- [Ubuntu 16.04主题美化和软件推荐(oh-my-zsh)](http://www.linuxidc.com/Linux/2016-09/135165p2.htm)
- [Ubuntu美化系列文章](http://www.jianshu.com/u/b3288a70ead9)
