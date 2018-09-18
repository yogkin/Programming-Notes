# Shell

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。
Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

- Shell脚本(shell script)是一种为shell编写的脚本程序
- Shell环境：
- Bourne Shell (/usr/bin/sh或/bin/sh)
- Bourne Again Shell (/bin/bash)
- C Shell (/usr/bin/csh)
- K Shell (/usr/bin/ksh)

Bash是大多数Linux 系统默认的 Shell。

---
## 1 运行 Shell 脚本

```Shell
#!/bin/bash
echo "Hello World !"
```

>"#!"是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种 Shell。

- 方法1：作为可执行程序

```Shell
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

- 方法2：作为解释器参数

```Shell
/bin/sh test.sh
/bin/php test.php
```

---
## 2 变量

### 定义变量

```Shell
your_name="runoob.com"
name=abc
```

- 变量名和等号之间不能有空格
- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头
- 中间不能有空格，可以使用下划线`_`
- 不能使用标点符号
- 不能使用bash里的关键字（可用help命令查看保留关键字）
- 已定义的变量，可以被重新定义

除了显式地直接赋值，还可以用语句给变量赋值：

```Shell
#将 /etc 下目录的文件名循环出来。
for file in `ls /etc`
```

### 使用变量

使用一个定义过的变量，只要在变量名前面加美元符号即可，建议给所有变量加上花括号

```Shell
#使用变量
your_name="qinjx"
echo $your_name
echo ${your_name}

# for循环中skill被赋值为被遍历的项
for skill in Ada Coffe Action Java; do
    echo "I am good at ${skill}Script"
done
```

### 只读变量

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变。

```Shell
#!/bin/bash
abc=123
readonly abc
```

### 删除变量

```Shell
unset variable_name
```

### 变量类型

1. 局部变量：局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
2. 环境变量：所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
3. shell变量：shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行

---
## 3 字符串

字符串可以用单引号，也可以用双引号，也可以不用引号

单引号字符串的限制：

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

双引号的优点：

- 双引号里可以有变量
- 双引号里可以出现转义字符

常见字符串操作：

```Shell
str='this is a string'

your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"

#拼接字符串
your_name="qinjx"
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

#获取字符串长度
string="abcd"
echo ${#string}

#提取子字符串
tring="runoob is a great site"
echo ${string:1:4}

#查找子字符串，查找字符 "i 或 s" 的位置
string="runoob is a great company"
echo `expr index "$string" is`
```

---
## 4 数组

bash支持一维数组，并且没有限定数组的大小。

- 定义数组：`数组名=(值1 值2 ... 值n)`
- 读取数组：`${数组名[下标]}`

```Shell
#定义数组
array_name=(value0 value1 value2 value3)

#单独赋值
array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen

#使用@符号可以获取数组中的所有元素
echo ${array_name[@]}

#获取数组的长度
length=${#array_name[@]}
length=${#array_name[*]}

#取得数组单个元素的长度
lengthn=${#array_name[n]}
```

---
## 5 Shell参数

在执行Shel 脚本时，可以向脚本传递参数(参数以空格隔开)，脚本内获取参数的格式为：`$n`

其他参数：
参数名|说明
---|---
`$#` | 传递到脚本的参数个数
`$*` | 以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
`$$` | 脚本运行的当前进程ID号
`$!` | 后台运行的最后一个进程的ID号
`$@` | 与`$*`相同，但是使用时加引号，并在引号中返回每个参数。如`$@`用引号`"`括起来的情况、以`"$1" "$2" … "$n"` 的形式输出所有参数。
`$-` | 显示Shell使用的当前选项，与set命令功能相同。
`$?` | 显示最后命令的退出状态。**0表示没有错误**，其他任何值表明有错误。比如：127表示没有找到命令；126表示命令不可执行

`$*` 与 `$@` 区别：

- 相同点：都是引用所有参数。
- 不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数`1、2、3`，则 `" * "` 等价于 `"1 2 3"`（传递了一个参数），而 `"@"` 等价于 `"1" "2" "3"`（传递了三个参数）。

```Shell
#!/bin/bash

echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i    #这里只输出一次
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i    #这里有几个参数就输出几次
done
```

---
## 6 运算符

Shell包括以下运算符：

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 `awk，expr`：

```Shell
#!/bin/bash

#expr的一个例子：求和
val=`expr 2 + 2`
echo  "两数之和为 : $val"
```

- 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成` 2 + 2 `。
- 完整的表达式要被 \`  \`符号 包含，这个字符不是常用的单引号，在数字键1的左边

### 6.1 算术运算符

假定变量 a 为 10，变量 b 为 20:

运算符|说明|示例
---|---|---
`+` | 加法 | `expr $a + $b` 结果为 30。
`-` | 减法 | `expr $a - $b` 结果为 -10。
`*` | 乘法 | `expr $a \* $b` 结果为 200。
`/` | 除法 | `expr $b / $a` 结果为 2。
`%` | 取余 | `expr $b % $a` 结果为 0。
`=` | 赋值 | `a=$b` 将把变量 b 的值赋给 a。
`==` | 相等。用于比较两个数字，相同则返回 true。| `[ $a == $b ]` 返回 false。
`!=` | 不相等。用于比较两个数字，不相同则返回 true。 | `[ $a != $b ]` 返回 true。

- 乘号`*`前边必须加反斜杠`\`才能实现乘法运算；
- 在 MAC 中 shell 的 expr 语法是：`$((表达式))`，此处表达式中的 `*` 不需要转义符号 `\` 。
- 条件表达式要放在方括号之间，并且要有空格，例如: `[$a==$b]` 是错误的，必须写成 `[ $a == $b ]`

### 6.2 关系运算符

假定变量 a 为 10，变量 b 为 20:

运算符 | 说明 | 示例
---|---|---
-eq | 检测两个数是否相等，相等返回 true。 | `[ $a -eq $b ]` 返回 false。
-ne | 检测两个数是否相等，不相等返回 true。 | `[ $a -ne $b ]` 返回 true。
-gt | 检测左边的数是否大于右边的，如果是，则返回 true。 | `[ $a -gt $b ]` 返回 false。
-lt | 检测左边的数是否小于右边的，如果是，则返回 true。 | `[ $a -lt $b ]` 返回 true。
-ge | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | `[ $a -ge $b ]` 返回 false。
-le | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | `[ $a -le $b ]` 返回 true。

- 关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

### 6.3 布尔运算符

假定变量 a 为 10，变量 b 为 20:

运算符 | 说明 | 示例
---|---|---
! | 非运算，表达式为 true 则返回 false，否则返回 true。 | `[ ! false ]`返回 true。
-o | 或运算，有一个表达式为 true 则返回 true。 | `[ $a -lt 20 -o $b -gt 100 ]` 返回 true。
-a | 与运算，两个表达式都为 true 才返回 true。 | `[ $a -lt 20 -a $b -gt 100 ]` 返回 false。

### 6.4 逻辑运算符

假定变量 a 为 10，变量 b 为 20:

运算符 | 说明 | 示例
---|---|---
`&&` | 逻辑的 AND | `[[ $a -lt 100 && $b -gt 100 ]]` 返回 false
`||` | 逻辑的 OR | `[[ $a -lt 100 || $b -gt 100 ]]` 返回 true

注意上面是**双方括号**

### 6.5 字符串运算符

假定变量 a 为 "abc"，变量 b 为 "efg"：

运算符 | 说明 | 示例
---|---|---
= | 检测两个字符串是否相等，相等返回 true。 | `[ $a = $b ]` 返回 false。
!= | 检测两个字符串是否相等，不相等返回 true。 | `[ $a != $b ]` 返回 true。
-z | 检测字符串长度是否为0，为0返回 true。 | `[ -z $a ]` 返回 false。
-n | 检测字符串长度是否为0，不为0返回 true。 | `[ -n $a ]` 返回 true。
str | 检测字符串是否为空，不为空返回 true。 | `[ $a ]` 返回 true。

### 6.6 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

加上`file="/var/www/runoob/test.sh"`


操作符 | 说明 | 举例
---|---|---
-b file | 检测文件是否是块设备文件，如果是，则返回 true。 | `[ -b $file ]` 返回 false。
-c file | 检测文件是否是字符设备文件，如果是，则返回 true。 | `[ -c $file ]` 返回 false。
-d file | 检测文件是否是目录，如果是，则返回 true。 | `[ -d $file ]` 返回 false。
-f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | `[ -f $file ]` 返回 true。
-g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。 | `[ -g $file ]` 返回 false。
-k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。 | `[ -k $file ]` 返回 false。
-p file | 检测文件是否是有名管道，如果是，则返回 true。 | `[ -p $file ]` 返回 false。
-u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。 | `[ -u $file ]` 返回 false。
-r file | 检测文件是否可读，如果是，则返回 true。 | `[ -r $file ]` 返回 true。
-w file | 检测文件是否可写，如果是，则返回 true。 | `[ -w $file ]` 返回 true。
-x file | 检测文件是否可执行，如果是，则返回 true。 | `[ -x $file ]` 返回 true。
-s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。 | `[ -s $file ]` 返回 true。
-e file | 检测文件（包括目录）是否存在，如果是，则返回 true。 | `[ -e $file ]` 返回 true。


---
## 7 命令

### 7.1 echo

用于字符串的输出，常用用法如下：

```Shell
#显示普通字符串
echo  "It is a test"
echo It is a test

#显示转义字符
echo  "\"It is a test\""

#显示变量，read 命令从标准输入中读取一行,并把输入行的每个字段的值指定给 shell 变量
read name
echo  "$name It is a test"

#显示换行
echo -e "OK! \n"  # -e 开启转义
echo  "It it a test"

#显示结果定向至文件
echo  "It is a test"  > file

#原样输出字符串，不进行转义或取变量(用单引号)
echo  '$name\"'

#显示命令执行结果，这里的符号是反引号 `
echo  `date`
```

总结：

- | 能否引用变量 | 能否引用转移符 | 能否引用文本格式符(如：换行符、制表符)
--- | --- | --- | ---
单引号 | 否 | 否 | 否
双引号 | 能 | 能 | 能
无引号 | 能 | 能 | 否

### 7.2 printf

printf 命令模仿 C 程序库（library）里的 printf() 程序。printf 由 POSIX 标准所定义，因此使用 printf 的脚本比使用 echo 移植性好。printf 使用引用文本或空格分隔的参数，外面可以在 printf 中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。默认 printf 不会像 echo 自动添加换行符，我们可以手动添加 `\n`。

```Shell
printf format-string [arguments...]
#format-string: 为格式控制字符串
#arguments: 为参数列表。
```

示例：

```Shell
printf  "%-10s %-8s %-4s\n" 姓名 性别 体重kg
```

### 7.3 test

Shell中的 test 命令用于检查某个条件是否成立，它可以进行**数值、字符和文件**三个方面的测试。

- 数值测试，使用关系运算符
- 字符串测试，使用字符串运算符
- 文件测试，使用文件测试运算符

示例：

```Shell
#数字
num1=100
num2=100
if  test $num1 -eq $num2
then
echo  '两个数相等！'
else
echo  '两个数不相等！'
fi

#字符串
str1="ru1noob"
str2="runoob"
if  test $str1 = $str2
then
echo  '两个字符串相等!'
else
echo  '两个字符串不相等!'
fi

#文件
if  test -e ./bash
then
echo  '文件已存在!'
else
echo  '文件不存在!'
fi

#多个条件使用( -a )、或( -o )、非( ! )
if  test -e ./notFile -o -e ./bash
then
echo  '至少有一个文件存在!'
else
echo  '两个文件都不存在'
fi
```

## 8 流程控制

sh的流程控制不可为空，比如如果else分支没有语句执行，就不要写这个else

### if

```Shell
#语法
if condition1
then
command1
elif condition2
then
command2
else
commandN
fi

#示例
a=10
b=20
if [ $a == $b ]
then
echo  "a 等于 b"
elif [ $a -gt $b ]
then
echo  "a 大于 b"
elif [ $a -lt $b ]
then
echo  "a 小于 b"
else
echo  "没有符合的条件"
fi
```

if的条件必须是命令，只有命令的退出状态码为0，then才会执行。

### for循环

for循环一般格式为：

```Shell
#多行
for  var  in item1 item2 ... itemN
do
command1
command2
...
commandN
done

#单行
for  var  in item1 item2 ... itemN;  do command1; command2… done;
```

示例：

```Shell
#一般用法
for  loop  in 1 2 3 4 5
do
echo  "The value is: $loop"
done

#类似其他语言的用法，通常情况下 shell 变量调用需要加 $,但是 for 的 (()) 中不需要
for((i=1;i<=5;i++));do
echo  "这是第 $i 次调用";
done;

#设置for的分隔符，IFS字段分隔符
list="windows--linux--macos"
IFS=$--
for  item  in $list
do
echo $item
done
```

### while 语句

```Shell
# 语法
while condition
do
command
done

#无效循环1
while  :
do
command
done

#无效循环2
while  true
do
command
done

# 示例
int=1
while(( $int<=5 ))
do
echo $int
let  "int++"
done
```

### until 循环

until循环执行一系列命令直至条件为真时停止。until循环与while循环在处理方式上刚好相反。

```Shell
until condition
do
command
done
```

### case

case语句为多选择语句。可以用case语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令

```Shell
#每一模式必须以右括号结束。取值可以为变量或常数，也可以是多个数的组合
#如果没有匹配模式，使用星号 * 捕获该值，再执行后面的命令
#每个模式已;;介绍，整个case以esac结束

#语法
case 值 in
模式1)
command1
command2
...
commandN
;;
模式2)
command1
command2
...
commandN
;;
*)
command1
;;
esac

# 示例：
echo -n "输入 1 到 5 之间的数字:"
read aNum
case $aNum in
1|2|3|4|5) echo  "你输入的数字为 $aNum!"
;;
*) echo  "你输入的数字不是 1 到 5 之间的! 游戏结束"
;;
esac
```

### 跳出循环

- break
- continue

---
## 9 函数

shell中可以定义函数，然后在shell脚本中可以调用。

格式：

```Shell
[ function  ] funname [()]

{

action;

[return int;]

}
```

- 可以带`function fun()` 定义，也可以直接`fun()` 定义，不带任何参数。
- 参数返回可以显式加：`return` 返回，如果不加，将以最后一条命令运行结果，作为返回值。
- 调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...

示例：

```Shell
function  fun(){
echo  "这是我的第一个 shell 函数!"
}

funWithReturn(){
echo  "这个函数会对输入的两个数字进行相加运算..."
echo  "输入第一个数字: "
read aNum
echo  "输入第二个数字: "
read anotherNum
echo  "两个数字分别为 $aNum 和 $anotherNum !"
return  $(($aNum+$anotherNum))
}

funWithParam(){
echo  "第一个参数为 $1 !"
echo  "第二个参数为 $2 !"
echo  "第十个参数为 $10 !"
echo  "第十个参数为 ${10} !"
echo  "第十一个参数为 ${11} !"
echo  "参数总数有 $# 个!"
echo  "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

---
## 10 输入输出重定向

大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回​​到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。

符号 | 说明
--- | ---
`command > file` | 将输出重定向到 file。
`command < file` | 将输入重定向到 file。
`command >> file` | 将输出以追加的方式重定向到 file。
`n > file` | 将文件描述符为 n 的文件重定向到 file。
`n >> file` | 将文件描述符为 n 的文件以追加的方式重定向到 file。
`n >& m` | 将输出文件 m 和 n 合并。
`n <& m` | 将输入文件 m 和 n 合并。
`<< tag` | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。

示例：

```Shell
#统计test.的行数
wc -l test.sh
# 输出：17 test.sh

#使用 < 的区别：把test.sh的内容输入给wc命令
wc -l < test.sh
#输出：17，没有文件名，因为wc此时只获取了内容

#把file的内容重定向到file2中
echo  < file1 > file2
```

### 标准输入文件

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
- 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
- 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

默认情况下，`command > file` 将 stdout 重定向到 file，`command < file` 将stdin 重定向到 file。

```Shell
#将 stderr 重定向到 file，可以这样写：
command 2 > file

#stderr 追加到 file 文件末尾
command 2 >> file

#将 stdout 和 stderr 合并后重定向到 file
command  > file 2>&1
command  >> file 2>&1
```

### Here Document

Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序。它的基本的形式如下：

```Shell
command  <<  delimiter
document
delimiter
```

- 它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command。
- 结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
- 开始的delimiter前后的空格会被忽略掉。

```Shell
#此时EOF是delimiter
wc -l <<  EOF
> hh
> ee
> cc
> EOF
3
```

### /dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 `/dev/null`，`/dev/null` 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 `/dev/null` 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。

```Shell
command  > /dev/null
#同时屏蔽错误输出
command  > /dev/null 2>&1
```

### 永久重定向
```Shell
exec 1>test.sh
```

---
## 11 文件包含

Shell可以包含外部脚本。这样可以很方便的封装一些通用的代码作为一个独立的脚本。Shell 文件包含的语法格式如下：

```Shell
#使用.引用文件，注意点号(.)和文件名中间有一空格
. filename
#使用source引用文件
source filename
```

实例：

```Shell
#使用 . 号来引用test1.sh 文件
. ./test1.sh

# 或者使用以下包含文件代码
source ./test1.sh
```

---
## 12 常用命令和工具

### wc

利用wc指令可以计算文件的**Byte数、字数、或是列数**，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。

```Shell
#语法
wc [-clw][--help][--version][文件...]
```

- -c或--bytes或--chars 只显示Bytes数。
- -l或--lines 只显示行数。
- -w或--words 只显示字数。
- --help 在线帮助。
- --version 显示版本信息。

### bc

bc命令 是一种支持任意精度的交互执行的计算器语言。bash内置了对整数四则运算的支持，但是并不支持浮点运算，而bc命令可以很方便的进行浮点运算，当然整数运算也不再话下。

语法：`bc(选项)(参数)`

选项：

- -i：强制进入交互式模式；
- -l：定义使用的标准数学库；
- -w：对POSIX bc的扩展给出警告信息；
- -q：不打印正常的GNU bc环境信息；
- -v：显示指令版本信息；
- -h：显示指令的帮助信息。

示例：

```Shell
echo  "1.212*3"  | bc
#设定小数精度（数值范围）
echo  "scale=2;3/8"  | bc
#将十进制转换成二进制
echo  "obase=2;$abc"  | bc
#将二进制转换成十进制
echo  "obase=10;ibase=2;$abc"  | bc
#幂运算
echo  "10^10"  | bc
#平方根
echo  "sqrt(100)"  | bc
```

### expr

expr命令 是一款表达式计算工具，使用它完成表达式的求值操作。

expr的常用运算符：

- 加法运算：`+`
- 减法运算：`-`
- 乘法运算：`*`
- 除法运算：`/`
- 求摸（取余）运算：`%`

示例：

```Shell
result=`expr 3 + 3`
result=$(expr $num1 + 25)
```

### read

read 命令一个一个词组地接收输入的参数，每个词组需要使用空格进行分隔；如果输入的词组个数大于需要的参数个数，则多出的词组将被作为整体为最后一个参数接收。

read参数：

- `-p` 输入提示文字
- `-n` 输入字符长度限制(达到指定位数，自动结束)
- `-t` 输入限时
- `-s` 隐藏输入内容

示例：

```Shell
read -p "请输入一段文字:" -n 6 -t 5 -s password
```

### let

let命令 是bash中用于计算的工具，提供常用运算符外还提供了方幂`**`运算符。

示例：

```Shell
#自加操作
let no++
#自减操作
let no--
#简写形式：等同于let no=no+10
let no+=10
#跟多个运算
let a=5+4 b=9-3
#多个运算，t1的值为最后一个表达式的值，这里为15-c-11，即与c相等
let  "t1 = ((a = 5 + 3, b = 7 - 1, c = 15 - 4))"
```

### exit

exit命令 同于退出shell，并返回给定值。在shell脚本中可以终止当前脚本执行。执行exit可使shell以指定的状态值退出。若不设置状态值参数，则shell以预设值退出。状态值0代表执行成功，其他值代表执行失败

---
## 13 `[]、()、[[]]、(())`

### 小括号 `( )`

- 命令组：括号中的命令将会新开一个子shell顺序执行，所以括号中的变量不能够被脚本余下的部分使用。括号中多个命令之间用分号隔开，最后一个命令可以没有分号，各命令和括号之间不必有空格。
- 命令替换：等同于`cmd`，shell扫描一遍命令行，发现了`$(cmd)`结构，便将`$(cmd)`中的cmd执行一次，得到其标准输出，再将此输出放到原来命令。有些shell不支持，如tcsh。
- 用于初始化数组。如：`array=(a b c d)`

### 双小括号 `(( ))`

- 整数扩展。这种扩展计算是整数型的计算，不支持浮点型。`((exp))`结构扩展并计算一个算术表达式的值，如果表达式的结果为0，那么返回的退出状态码为1，或者是"假"，而一个非零值的表达式所返回的退出状态码将为0，或者是"true"。若是逻辑判断，表达式exp为真则为1,假则为0。
- 只要括号中的运算符、表达式符合C语言运算规则，都可用在`$((exp))`中，甚至是三目运算符。作不同进位(如二进制、八进制、十六进制)运算时，输出结果全都自动转化成了十进制。如：`echo $((16#5F))` 结果为95 (16进位转十进制)
- 单纯用 `(( ))` 也可重定义变量值，比如 `a=5; ((a++))` 可将 `$a` 重定义为6
- 常用于算术运算比较，双括号中的变量可以不使用`$`符号前缀。括号内支持多个表达式用逗号分开。 只要括号中的表达式符合C语言运算规则，比如可以直接使用`for((i=0;i<5;i++))`, 如果不使用双括号, 则`for i in {0..4}`或者下面表达式。再如可以直接使用`if (($i<5))`, 如果不使用双括号, 则为`if [ $i -lt 5 ]`。

```Shell
for  i  in  `seq 0 4`
```

### 单中括号 `[]`

- bash的内部命令：`[`和test是等同的。如果我们不用绝对路径指明，通常我们用的都是bash自带的命令。`if/test`结构中的左中括号是调用`test`的命令标识，右中括号是关闭条件判断的。这个命令把它的参数作为比较表达式或者作为文件测试，并且根据比较的结果来返回一个退出状态码。`if/test`结构中并不是必须右中括号，但是新版的Bash中要求必须这样。
- test和`[]`中可用的比较运算符只有`==`和`!=`，两者都是用于字符串比较的，不可用于整数比较，整数比较只能使用`-eq`，`-gt`这种形式。无论是字符串比较还是整数比较都不支持大于号小于号。如果实在想用，对于字符串比较可以使用转义形式，如果比较"ab"和"bc"：`[ ab \< bc ]`，结果为真，也就是返回状态为0。`[ ]`中的逻辑与和逻辑或使用-a 和-o 表示。
- 字符范围：用作正则表达式的一部分，描述一个匹配的字符范围。作为test用途的中括号内不能使用正则。
- 在一个array 结构的上下文中，中括号用来引用数组中每个元素的编号。

```Shell
if [ $var -eq 0 ];  then  echo  "True";  fi
#等价于
if  test $var -eq 0;  then  echo  "True";  fi
```

### 双中括号 `[[ ]]`

- `[[`是 bash 程序语言的关键字。并不是一个命令，`[[ ]]` 结构比`[ ]`结构更加通用。在`[[`和`]]`之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
- 支持字符串的模式匹配，使用`=~`操作符时甚至支持shell的正则表达式。字符串比较时可以把右边的作为一个模式，而不仅仅是一个字符串，比如`[[ hello == hell? ]]`，结果为真。`[[ ]]` 中匹配字符串或通配符，不需要引号。
- 使用`[[ ... ]]`条件判断结构，而不是`[ ... ]`，能够防止脚本中的许多逻辑错误。比如，`&&、||、<和>` 操作符能够正常存在于`[[ ]]`条件判断结构中，但是如果出现在`[ ]`结构中的话，会报错。比如可以直接使用`if [[ $a != 1 && $a != 2 ]]`, 如果不适用双括号, 则为`if [ $a -ne 1] && [ $a != 2 ]`或者`if [ $a -ne 1 -a $a != 2 ]`。
- bash把双中括号中的表达式看作一个单独的元素，并返回一个退出状态码。

---
## 14 数学计算

数学计算总结
```Shell
# let方式
let result+=5
# []操作符方式
result=$[ nb1+nb2 ]
result=$[ nb1 + 10 ]
# (())方式
$(($nb1+$nb2))
# expr方式
$(expr 3 + 4)
$(expr $nb1 + 10)
# bc方式
result=`echo "$nb3 * 1.5" | bc`
```
---
## 引用

- [Linux 命令大全](http://www.runoob.com/linux/linux-command-manual.html)
- [Shell教程](http://www.runoob.com/linux/linux-shell.html)
- [shell中各种括号的作用`()、(())、[]、[[]]、{}`](http://blog.csdn.net/taiyang1987912/article/details/39551385)