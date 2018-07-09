# 二进制 I/O

把数据写到文件效率最高地方法是用二进制形式写入。二进制输入避免了在数值转换为字符串过程中所涉及地开销和精度损失。但二进制数据并非人眼所能阅读，所以这个技巧只有当数据将被另一个程序按顺序读取时才能使用。

`fread` 函数用于读取二进制数据， `fwrite` 函数用于写入二进制数据。它们的原型如下所示：

```c++
size_t fread(void * buffer, size_t size, size_t count, FILF * stream);
size_t fwrite(void * buffer, size_t size, size_t count, FILF * stream);
```

buffer 是一个指向用于保存数据的内存位置的指针， size 是缓冲区中每个元素的字节数， count 是读取或写入的元素数， stream 是数据读取或写入的流。

buffer 参数被解释为一个或多个值的数组。 count 参数指定数组中值的个数，所以读取或写入一个标量时， count 的值应为 1 。函数的返回值是实际读取或写入的元素（并非字节）数目。如果输入过程遇到了文件尾或者输出过程中出现了错误，这个数字可能比请求的元素数目要小。

下面是一段代码示例：

```c++
struct VALUE {
    long a;
    float b;
    char c[size];
} values[ARRAY_SIZE];
...
n_values = fread(values, sizeof(struct VALUE), ARRAY_SIZE, input_stream);
...
fwrite(values, sizeof(struct VALUE), n_values, output_stream);
```

这段代码从一个输入文件读取二进制数据，对数据进行处理之后把结果再写入到一个输出文件中。前面提到过，这种类型的 I/O 效率很高，因为每个值中的位直接从流读取或向流写入，不需要任何转换。
