# typedef

---
## typedef 的作用

typedef可以指定新的类型名来代替已有的类型名，目的是为了使源代码更易于阅读和理解。习惯把typedef声明的类型的第一次字母大写。

定义一个新的类型的方法是：

1. 先按定义变量的方法写出定义体(int a)
2. 将变量名换成新的类型名(Count)
3. 在最前面加上typedef(typedef Count)
4. 然后可以用新的类型名去定义变量

```c
typedef int Integer//给int定义一个Integer名称。
typedef void (*__p_sig_fn_t)(int);//定义一个函数指针。
```

---
## typedef和#define的区别

把typedef 看成一种彻底的“封装”类型，声明之后不能再往里面增加别的东西。

- 1 可以使用其他类型说明符对宏类型名进行扩展，但对 typedef 所定义的类型名却不能这样做。
```c
#define INTERGE int
unsigned INTERGE n; //没问题

typedef int INTERGE;
unsigned INTERGE n; //错误，不能在 INTERGE 前面添加 unsigned
```
- 2 在连续定义几个变量的时候，typedef 能够保证定义的所有变量均为同一类型，而 #define 则无法保证。
```
#define PTR_INT int *
PTR_INT p1, p2;
int *p1, p2;//经过宏替换以后

typedef int * PTR_INT
PTR_INT p1, p2//p1、p2 类型相同，它们都是指向 int 类型的指针。
```
- 3 typedef与define表面上有相似之处，但是本质不一样，define在预编译时处理，而typedef在编译时处理。