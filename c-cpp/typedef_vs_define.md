# typedef与\#define对比

在 `typedef` 和宏文本替换之间存在着关键性的区别。正确思考这个问题的方法应该是把 `typedef` 看成是一种彻底的“封装”类型——在声明它之后不能再往里面增加别的东西。它和宏的区别主要体现在两个方面。

首先，可以用其他类型说明符对宏类型名进行扩展，但对 `typedef` 所定义的类型名却不能这样做。如下所示：

```c++
#define A int
unsigned A i; // 没问题
typedef int B;
unsigned B j; // 错误！非法！
```

其次，在连续的几个变量的声明中，用 `typedef` 定义的类型能够保证声明中所有的变量均为同一类型，而用 `#define` 定义的类型则无法保证。如下所示：

```c++
#define int_ptr int *
int_ptr i, j;
```

经过宏扩展，第二行变为：

```c++
int * i,j;
```

这使得 `i` 和 `j` 成为不同的类型—— `i` 是一个指向 `int` 的指针，而 `j` 则是一个 `int` 变量。相反，下面的代码中：

```c++
typedef char * char_ptr;
char_ptr m,n;
```

`m` 和 `n` 的类型依然相同。虽然前面的类型名变了，但它们的类型相同，都是指向 `char` 的指针。
