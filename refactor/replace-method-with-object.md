# 以对象取代函数

对于一个大型函数，其中对局部变量的使用导致无法很好地提取函数时，可以将这个函数放进一个单独对象中，如此一来局部变量就成了对象内的字段，然后就可以在同一对象中将这个大型函数分解成若干小型函数了。

### 做法

1. 建立一个新类，根据待处理函数的用途为这个新类命名。
2. 针对原函数的每个临时变量和每个参数，在新类中建立一个对应的字段保存。
3. 在新类中建立一个构造函数，接收原函数的所有参数作为参数。
4. 在新类中建立一个公有函数。
5. 将原函数的实现源码复制到上一步建立的公有函数中。
6. 将原函数的代码替换为对新对象中公有函数的调用。
7. 编译、运行、调试。
8. 对新函数执行[提炼函数](./extract-method.md)操作。

### 范例

```c++
class Account
{
    public:
        int gamma(int in_val0, int in_val1, int in_val2)
        {
            int tmp_val0 = in_val0 * in_val1 + 10;
            int tmp_val1 = in_val0 * in_val2 + 100;
            if (in_val2 - tmp_val0 > 100)
                tmp_val1 -= 20;
            int tmp_val2 = tmp_val1 * 7;
            return tmp_val2 - 2 * tmp_val0;
        }
};
```

声明新类，保存原函数的每个临时变量和每个参数：

```c++
class Gamma
{
    private:
        int _in_val0;
        int _in_val1;
        int _in_val2;
        int _tmp_val0;
        int _tmp_val1;
        int _tmp_val2;
};
```

加入一个构造函数：

```c++
Gamma(int in_val0, int in_val1, int in_val2)
{
    this->_in_val0 = in_val0;
    this->_in_val1 = in_val1;
    this->_in_val2 = in_val2;
    this->_tmp_val0 = this->_tmp_val1 = this->_tmp_val2 = 0;
}
```

将源函数的代码搬到`Gamma`类的公有函数中：

```c++
class Gamma
{
    public:
        int compute()
        {
            this->_tmp_val0 = this->_in_val0 * this->_in_val1 + 10;
            this->_tmp_val1 = this->_in_val0 * this->_in_val2 + 100;
            if (this->_in_val2 - this->_tmp_val0 > 100)
                this->_tmp_val1 -= 20;
            this->_tmp_val2 = this->_tmp_val1 * 7;
            return this->_tmp_val2 - 2 * this->_tmp_val0;
        }
};
```

替换源函数的实现主体：

```c++
class Account
{
    public:
        int gamma(int in_val0, int in_val1, int in_val2)
        {
            return (new Gamma(in_val0, in_val1, in_val2))->compute();
        }
};
```

此时，便可以对新函数执行[提炼函数](./extract-method.md)操作了。