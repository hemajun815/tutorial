# 从 `(*(void(*)())0)();` 理解函数声明

当看到 `(*(void(*)())0)();` 这样的表达式，每个程序员的内心都“不寒而栗”。然而，我们大可不必对此望而生畏。

任何 C 变量的声明都由两个部分组成：类型以及一组类似表达式的声明符。声明符从表面上看与表达式有些类似，对它求值应该返回一个声明中给定类型的结果。

最简单的声明符就是单个变量，如： `float f, g;` 。这个声明的含义是：当对其求值时，表达式 `f` 和 `g` 的类型为浮点类型。

因为声明符与表达式的相似，所以我们也可以在声明符中任意使用括号： `float ((f));` 。这个声明的含义是：当对其求值时， `((f))` 的类型为浮点类型，由此可以推知， `f` 也是浮点类型。

同样的逻辑也适用于函数和指针类型的声明，例如： `float ff();` 。这个声明的含义是：表达式 `ff()` 求值结果是一个浮点数，也就是说， `ff` 是一个返回值为浮点类型的函数。

类似地， `float *fp;` 。这个声明的含义是 `*fp` 是一个浮点数，也就是说， `fp` 是一个指向浮点数的指针。

以上这些形式在声明中还可以组合起来，就像在表达式中进行组合一样，因此 `float *g(), (*h)();` 表示 `*g()` 与 `(*h)()` 是浮点表达式。因为 `()` 结合优先级高于 `*` ， `*g()` 也就是 `*(g())` : `g` 是一个函数，该函数的返回值类型为指向浮点数的指针。同理，可以得出 `h` 是一个函数指针， `h` 所指向函数的返回值为浮点类型。

现在我们已经知道了如何声明一个给定类型的变量，那么该类型的类型转换符就很容易得到了。由于 `float (*h)();` 表示 `h` 是一个指向返回值为浮点类型的函数的指针，那么 `(float (*)())` 则表示一个“指向返回值为浮点类型的函数的指针”的类型转换符。

拥有了这些知识，我们现在可以分两步来分析表达式 `(*(void(*)())0)();` 。

第一步，假定变量 `fp` 是一个函数指针，那么如何调用 `fp` 所指向的函数呢？调用方式如下： `(*fp)();` 因为 `fp` 是一个函数指针，那么 `*fp` 就是该指针指向的函数，所以 `(*fp)();` 就是调用该函数的方式。

在表达式 `(*fp)();` 中， `*fp` 两侧的括号非常重要，因为函数运算符 `()` 的优先级高于单目运算符 `*` 。如果 `*fp` 两侧没有括号，那么 `*fp()` 实际上与 `*(fp())` 的含义完全一致。

现在，剩下的问题就只是找到一个恰当的表达式来替换 `fp` 。这就是我们第二步要做的事情。

假如 C 编译器能够理解我们大脑中对于类型的认识，那么我们可以这样写： `(*0)();` 。但是这并不会生效，因为运算符 `*` 必须要一个指针来做操作数。而且，这个指针还应该是一个函数指针，这样经运算符 `*` 作用后的结果才能作为函数被调用。因此，在上式中必须对 `0` 作类型转换，转换后的类型可以大致描述为：“指向返回值为 void 类型的指针”。

如果 `fp` 是一个指向返回值为 void 类型的函数的指针，那么 `(*fp)()` 的值就为 void，`fp`的声明为： `void (*fp)();` ，因此我们可以用下面的式子来完成调用存储位置为 0 的子程序：

```c++
// 这里假设 fp 默认初始值为 0, 不推荐这种写法。
void (*fp)();
(*fp)();
```

这种写法的代价是多声明了一个“哑”变量。

但是，我们一旦知道如何声明一个变量，也就自然知道如何对一个常数进行类型转换，将其转换成该变量的类型：只需要在变量声明中将变量名去掉即可。以此，将常数 0 转换成“指向返回值为 void 的函数的指针”类型，可以这样写： `(void(*)())0` 。因此，我们将用 `(void(*)())0` 替换 `fp` ，从而得到 `(*(void(*)())0)();` 。末尾的分号使得表达式成为一个语句。

---

参考文献：[《C陷阱与缺陷》](https://book.douban.com/subject/2778632/)
