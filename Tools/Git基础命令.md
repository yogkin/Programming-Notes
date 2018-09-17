
# Git 常用命令总结


---
## Git特点

- 分布式：各个Repo都有完整的镜像
- 快照：GIt每次记录的都是完整的Repo信息，而不是每个版本之间的差异
- 状态区：了解Git的状态区是学习git的重要步骤
- 分支：分支是Git最重要的功能之一

---
## 提交流程

提交流程：工作区(编辑文件)-->缓冲区(add)-->本地仓库(commit)-->远程仓库(push)

    master：表示默认的开发分支
    origin：默认的远程仓库名称
    HEAD：指向的是当前分支
    HEAD^：Head的父提交

---
## 配置



命令|说明
---|---
`git config --global user.name "name"`    | 配置用户名
`git config --global user.email "email"` | 配置邮箱
`git config --gobal core.autocrlf false`  | 系统的换行符错误
`git config --list --global`    |      查看全局信息
`git config --list`    |      查看所有配置信息
`git config user.name`     |     查看用户名称
`git config --global --unset user.name username`     |     删除一个配置
`git version`  | 查看版本


所有的配置都在.git隐藏文件夹内存config文件中




---
## 创建仓库

```
    git init
```

---
## 添加与提交


命令|说明
---|---
`git add file-name`    |      添加文件
`git commit -m`      |     "提交说明"
`git commit -a -m "提交说明"`   |       跳过缓冲区，把当前工作区已经添加到版本库的修改过的文件一次性添加到缓冲区，然后提交到版本库
`git commit --amend -m "提交说明"`      |   追加修改



---
## 查看

查看当前分支状态

命令|说明
---|---
`git status`    |      查看工作区状态
`git diff`       |   查看当前工作区与缓冲区不同的地方
`git diff HEAD -- <file-name>`  |      查看某一个文件当前版本和工作区有什么不同（注意空格）
`git diff --staged`      |    查看当前工作区与最后一次版本库不同的地方
`git shortlog`    |      根据提交者的名字进行分组


---
## Log

命令|说明
---|---
`git log`  |  查看每次提交           -p 选项展开显示每次提交的内容差异，
`git log -1`  | 使用 git log -1 表示只显示最近一次
`git log --status`    |   仅显示简要的增改行数统计
`git log --pretty=oneline`    |   --pretty 选项，可以指定使用完全不同于默认格式的方式展示提交历史。比如用 oneline 将每个提交放在一行显示，这在提交数很大时非常有用。另外还有 short，full 和 fuller 可以用，展示的信息或多或少有些不同
`git blame file_name`     |        追溯一个文件的历史修改记录
`git log --graph` |   git提交的log视图
`git log --pretty=format:"%h %s" --graph`    |   分支及其分化衍合情况
`git log  --graph --pretty=oneline --abbrev-commit` | 查看分支合并图
`git reflog`            | 得到所有提交记录，即使是已经被回退的版本


---
## 版本回滚与文件管理



命令|说明
---|---
`git reset --hard HEAD^`    |       回到上一个版本
`git reset --hard <commitId>` |     回到某一个版本
`git reset HEAD <file-name>`    |    撤销文件在缓冲区的修改
`git checkout -- <file-name>`    |    撤销工作区的修改
`git rm <file-name>`        |        删除文件，省去了add的操作
`git rm --cached <file-name>` |    文件已经添加到版本库，需要从版本库移除，但是不删除本地文件
`git mv README.md README`     |    移动文件 相当于下面的操作          
`git rm README.txt`        |        删除原来的版本库中的文件
`git add README`            |        新文件添加到版本库
`mv README.txt README`        |    重命名和移动

**reset参数说明：**

`--soft`：将回退的 commit 中所作的一些修改放在缓存区，不会导致修改丢失
`--mixed`：会将那些修改放回工作区，不在缓存区
`--hard`：会将工作区以及缓存区和历史区都回退到特定的commit状态，会导致修改丢失


---
## SSH KEY

    ssh-keygen -t rsa -C "your email address"     在`C:\Users\Administrator\.ssh`生成私匙，邮箱地址使用初始化的邮箱  (提醒ssh-keygen没有空格)
    ssh -T git@gitHub.com 验证ssh key


---
## 远程仓库



命令|说明
---|---
`git fetch <repository名称>`  | 从远程查看抓取数据，不合并
`git clone git_address`   |   克隆远程仓库并且自动关联
`git remote add <origin> <git@github.com:Ztiany/studyGit.github>` |  关联远程仓库,如果是在本地init的git仓库
`git pull origin master`  |  拉取分支并合并，一定要指定分支名
`git push -u origin master`  | 推送到远程仓库，并建立关联 （远程仓库是空的）
`git push origin master` | 远程仓库没有此分支则推送此分支，否则就是推送提交的内容
`git pull origin master ----allow-unrelated-histories`| 拉取分支，允许合并无相关的历史
`git push origin --delete <branch-name>`        |   删除远程分支
`git checkout -b <branch-name> <origin>/<branch-name>`  |   在本地创建和远程分支对应的分支 前提是远程仓库有这个分支
`git remote rm paul` | 删除远程paul仓库
`git push -f`|强制推送


关于`git pull --rebase`:

        开发者A将push修改到Repo时，开发者B已经push了自己的修改，
        这时候A需要先pull最新的修改，但这样会在Git历史中留下一个Merge History，
        使用`git pull --rebase`指令拉取最新的修改，该指令的作用是拉取本地代码后，
        将本地代未提交的代码作用到最新的版本中，从而避免多余的Merge History


---
## 分支

切换远程分支的时候先git fetch

命令|说明
---|---
`git branch`            |        查看分支
`git branch <name>`      |        新建分支
`git checkout <file>`     |    切换分支
`git checkout -b <name>` |        新建并且切换
`git branch -d <name>` |        删除
`git branch -D <name>`     |    强行删除未合并的分支



---
## 合并

命令|说明
---|---
`git merge <name>`    |   把指定分支合并到当前分支，Fast-forward信息，合并是“快进模式”，也就是直接把master指向dev的当前提交，
`git merge --no-ff -m "合并说明" <name>` |    禁用快速合并,加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
`git rebase master`           |   变基



**merge的说明**:

![](images/git_01_branchs.png)


把两个分支最新的快照（C3 和 C4）以及二者最新的共同祖先（C2）进行三方合并，合并的结果是产生一个新的提交对象（C5）

**rebase的说明** :

rebase就是变基的意思，假设master是主分支，dev是某个测试分支，当测试完基本功能后接受到master分支owner的通知，说master已经有了改动，所有的局部功能需要在新的master状态，对于dev分支来讲，就需要对自己当前的的分支**改变基点**，操作如下：

     git checkout dev
     git rebase master //以master为基点变基

使用rebase操作后的Git时间线会被进行合并，而Merge不会，Merge操作保持了完整的提交记录，而rebase让时间线变得更加干净，rebase一般是临时性的合并，而merge则是大功能的合并。




---
## 工作区stash

命令|说明
---|---
`git stash`         |    储藏工作区
`git stash list`         |   查看储藏
`git stash apply` |       恢复工作区
`git stash drop`    |    删除储藏
`git stash pop`        |    恢复并删除储藏


---
## 查看远程仓库

命令|说明
---|---
`git remote`                     |                    查看远程仓库信息
`git remote -v`                    |                详细信息
`git remote show <remoteName>`             |        查看指定仓库信息
`git remote rename <oldname> <newname>`         |    远程查看重命名,注意，对远程仓库的重命名，也会使对应的分支名称发生变化，原来的 pb/master 分支现在成了 paul/master。>


---
## TAG

命令|说明
---|---
`git tag`                 |            查看tag
`git tag name`             |            打tag
`git tag name commitId`             |    根据id打tag
`git tag name -m "tag说明" commitId` |    打tag时添加说明
`git push origin tag-name`        |    推送tag到远程仓库
`git push origin --tags`        |        推送本地所有tag
`git tag -d name`         |            删除tag
`git push origin :refs/tags/tagName`  |    删除远程tag(需要先删除本地tag)


---
## Git子模块

子模块就是你的一个Git项目需要应用其他Git项目，但是希望子模块能独立关联远程仓库。

    `git submodule add <git://github.com/chneukirchen/rack.git rack>` 在主项目下使用此命令关联子模块，如果操作成功，会在根目录生成.gitmodules文件

如果拉取了一个含有子模块的项目，需要执行以下命令，首先拉取住主项目，命令略

    `git submodule init` 初次拉取必须使用此目录初始化子模块
    `git submodule update` 然后更新子模块，主模块更新不会自动同步子模块，每次需要同步子模块都需要子命令
    
详情查看[文档](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

---
## 快捷配置

```
    git config --global color.ui.true
    git config --global alias.st status     配置别名
    git config --global alias.unstage 'reset HEAD'  配置别名  之后用git unstage file
    git config --global alias.last 'log -1'        配置别名  之后用git last
    git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

    git config --global core.quotepath false 解决log中文文件名乱码问题

    //利用 shell 配置带参数的别名
    p = "!f() { git push origin HEAD:refs/for/\"$1\"; }; f"
    file = "!f() { git diff --name-status \"$1\"; }; f"
```

---
## 文件匹配符号

    dir/ 表示目录
    dir/\*.class  表示dir下面的所有.class 结尾的文件

实例:          

    git rm log/\*.log 删除log目录下所有 ".log" 结尾的文件
    git rm \*~ 递归删除当前目录及其子目录下的所有 "~" 结尾的文件

---
## 技巧

```
    提示命令
    git st 忘了命令，连续两次tag，会有提示
```
---
## 忽略文件

    在.gitignore中编辑

---
## 冲突

```
            CONFLICT 冲突
            <<<<<<< HEAD //表示当前分支内容
            Creating a new branch is quick & simple.
            =======
            Creating a new branch is quick AND simple.
            >>>>>>> feature1 //表示被合并分支的内容
```

---
## 分支策略

1. 首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
2. 干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本

git开发流程图如下:

![](images/git_02_dev.png)

---
## GitHub保持与Forked的项目同步

```
git remote -v //查看当前的远程仓库地址
git remote add upstream 原来的仓库地址 //添加一个远程参考地址
git fetch upstream //拉取原仓库
git checkout master //切换到master
git merge upstream/master //合并
git push //推送到forked的仓库
```

---
## 清空所有提交记录

[how to delete all commit history in github? [duplicate]](https://stackoverflow.com/questions/13716658/how-to-delete-all-commit-history-in-github)

```
1.  Checkout
    `git checkout --orphan latest_branch`
2.  Add all the files
    `git add -A`
3.  Commit the changes
    `git commit -am "commit message"`
4.  Delete the branch
    `git branch -D master`
5.  Rename the current branch to master
    `git branch -m master`
6.  Finally, force update your repository
    `git push -f origin master`
```

---
## 修改远程仓库地址

```
git remote set-url origin [url]
```

---
## 资料

- [git.oschina ](http://git.oschina.net/progit/1-起步.html#1.5-%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C-Git-%E5%89%8D%E7%9A%84%E9%85%8D%E7%BD%AE)
- [史上最浅显易懂的Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
- [Git Community Book 中文版]( http://gitbook.liuhui998.com/ )
