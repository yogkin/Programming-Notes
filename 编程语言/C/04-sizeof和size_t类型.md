# sizeof和size_t类型

```c
sizt_t intSize;
sizt = sizeof(int);
printf("iniSize=%zd",intSize);
```

C语言规定，sizeof 返回的类型是 size_t，这是一个无符号的整数类型，由 typedef 定义，根据系统的不同，所使用整型也是不一样的。

C99做了进一步的调整，新增了使用`$zd`转换说明用于`printf()`显式 size_t 类型，如果系统不支持，可以使用`%u`或者`&lu`代替。

