# pdb

pdb是基于命令行的调试工具，非常类似gnu的gdb（调试c/c++）

| 命令 | 简写命令 | 作用 |
| --- | --- | --- |
| break | b | 设置断点 |
| continue | c | 继续执行程序 |
| list | l | 查看当前行的代码段 |
| step | s | 进入函数 |
| return | r | 执行代码直到从当前函数返回 |
| quit | q | 中止并退出 |
| next | n | 执行下一行 |
| print | p | 打印变量的值 |
| help | h | 帮助 |
| args | a | 查看传入参数 |
 回车 | 重复上一条命令 |
| break | b | 显示所有断点 |
| break lineno | b lineno | 在指定行设置断点 |
| break file:lineno | b file:lineno | 在指定文件的行设置断点 |
| clear num | 删除指定断点 |
| bt | 查看函数调用栈帧 |

执行时调试
```
python -m pdb some.py
```

交互调试
```
import pdb
pdb.run('testfun(args)') # 此时会打开pdb调试，注意：先使用s跳转到这个testfun函数中，然后就可以使用l看到代码了
```

程序里埋点
```python
# 当程序执行到pdb.set_trace() 位置时停下来调试
import pdb 
pdb.set_trace() 
```