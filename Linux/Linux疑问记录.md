# Linux 疑问记录

---
## 1 `/etc/shells` 文件用途

`/etc/shells` 是一个有效登陆shell的列表，在调用 chsh 改变登陆 shell 时，会查询这个文件。如果登录的 shell 不在 `/etc/shells` 里，普通用户调用 chsh 失败。root 调用会出现 warning。另外一些程序会根据这个文件来判断一个用户是否是有效用户，例如 FTP 服务会阻止那些 shell 不在 /etc/shells 里的用户登陆。

---
## 2 Linux用户管理

Linux用户管理最重要的两个文件就是 `/etc/passwd` 和 `/etc/shadow` 这两个文件。其中 `/etc/passwd` 是用来存储登陆用户信息的，它的基本格式如下：

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
```

由上面的格式可以看出，每一行代表一个用户的信息，一共包括 7 个字段的信息，每个字段的信息用冒号隔开。这 7 个字段分别代表：

1. 账号名称：即登陆时的用户名
2. 密码：早期UNIX系统的密码是放在这个文件中的，但因为这个文件的特性是所有程序都能够读取，所以，这样很容易造成数据被窃取，因此后来就将这个字段的密码数据改放到 `/etc/shadow`中
3. UID：用户ID，每个账号名称对应一个UID，通常 `UID=0` 表示root管理员
4. GID：组ID，与 `/etc/group` 有关，`/etc/group` 与 `/etc/passwd` 差不多，是用来规范用户组信息的
5. 用户信息说明栏： 用来解释这个账号是干什么的
6. Home目录：即用户登陆以后跳转到的目录，以root用户为例，`/root` 是它的家目录，所以 root 用户登陆以后就跳转到 `/root` 目录这里
7. Shell：用户使用的shell，通常使用 `/bin/bash` 这个 shell，这也就是为什么登陆Linux时默认的shell是bash的原因，就是在这里设置的，如果要想更改登陆后使用的shell，可以在这里修改。

---
## 3 特殊的 shell：`/sbin/nologin`

使用了这个 shell 的用户即使有了口令，进行登陆操作时也无法登陆，如果尝试登录会出现如下的信息：

    This account is currently not available

所谓的**无法登陆**指的仅是：这个使用者无法使用 bash 或其他 shell 来登陆系统， 并不是说这个账号就无法使用其他的系统资源。

如果我想要让某个具有 /sbin/nologin 的使用者知道，他们不能登入主机时，可以创建`/etc/nologin.txt`文件， 并且在这个档案内说明不能登录的原因，那么下次当这个用户想要登入系统时， 屏幕上出现的就会是 `/etc/nologin.txt` 这个档案的内容，而不是预设的内容了。

nologin 的作用就是限制某些用户通过 ssh 登陆到 shell 上。有时候为了进行系统维护工作，临时禁止其他用户登录，可以使用 nologin 文件，具体做法是在/etc/目录下创建一个名称为 nologin 的文件。例如：

```
touch /etc/nologin
echo disable login by admin temperarily! > etc/nologin
```

当用户试图登陆时，将会给用户显示 "disable login by admin temperarily!"，当系统维护结束以后，再删除 `/etc/nologin` 文件，其他用户就又可以恢复登陆了，这只是限于能登陆 shell 的用户来说的，对于那些登陆shell为 `/sbin/nologin` 的用户来说没有影响，因为他们本身就无法登陆shell。


具体参考：

- [第十四章、Linux 账号管理与 ACL 权限配置](http://cn.linux.vbird.org/linux_basic/0410accountmanager.php#nologin)
- [linux中/etc/nologin文件的作用](https://blog.csdn.net/zhouhuakang/article/details/51168104)