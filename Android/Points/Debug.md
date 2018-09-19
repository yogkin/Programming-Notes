# 从Debug看方法执行流程

开发中经常用Debug调试程序，用来查找各种bug，但是Debug还可以看当前虚拟机有几个线程在运行，并且还可以查看每个线程执行的堆栈信息，这对理清代码执行流程很有帮助，比如假如我们不知道App的启动入口是ActivityThread，也没有资料可以用来查询，那么可以用Debug来查看main线程的堆栈信息：

- 打断点
![](index_files/b271246e-d18e-40f9-9aca-d56c5fce5680.png)

- 查看main线程的堆栈信息，可以看到Zygote和ActivityThread
![](index_files/3238d2a9-552c-424e-ae6a-9a3ea42cf04d.png)

- 切换线程
![](index_files/de2bcc8a-4f82-4d1e-b932-e8a36d7b5d93.png)