# 提炼函数

将一段可以被组织在一起并独立出来的代码放进一个函数中，并让函数的名称解释该函数的用途。

如：

```c++
void printOwing(float amount)
{
    printBanner();

    // print details
    std::cout << "name: " + _name << std::endl;
    std::cout << "amount: " + std::to_string(amount) << std::endl;
}
```

转变为：

```c++
void printOwing(float amount)
{
    printBanner();
    printDetails(amount);
}

void printDetails(float amount)
{
    std::cout << "name: " + _name << std::endl;
    std::cout << "amount: " + std::to_stringamount) << std::endl;
}
```

### 做法

1. 创建一个新函数，根据函数的意图对其命名，反应其“做什么”，而非“怎么做”。
2. 将提炼出的代码从源函数复制到新建的目标函数中。
3. 检查提炼出的代码，看看其中是否引用了“作用域源于源函数”的变量（包含局部变量和源函数参数）。
4. 检查是否有“仅用于被提炼代码段”的临时变量。如果有，在目标函数中将其声明为临时变量。
5. 检查被提炼代码段，看是否有任何局部变量的值被改变。如果一个临时变量值被修改了，看看是否可以将被提炼的代码段处理为一个查询，并将结果赋值给相关变量。
6. 将被提炼代码段中需要读取的局部变量当作参数传递给目标函数。
7. 整理完所有局部变量后，进行编译。
8. 在源函数中，将被提炼的代码段替换为对目标函数的调用。
9. 编译、运行、测试。

### 范例一：无局部变量

```c++
void printOwing()
{
    // print banner
    std::cout << "*************************" << std::endl;
    std::cout << "***** Customer Owes *****" << std::endl;
    std::cout << "*************************" << std::endl;

    // calculate outstanding
    auto outstanding = 0.f;
    for (auto each : _orders)
    {
        outstanding += each.getAmount();
    }

    // print details
    std::cout << "name: " + _name << std::endl;
    std::cout << "amount: " + std::to_string(outstanding) << std::endl;
}
```

提炼“print banner”部分代码：

```c++
void printOwing()
{
    printBanner();

    // calculate outstanding
    auto outstanding = 0.f;
    for (auto each : _orders)
    {
        outstanding += each.getAmount();
    }

    // print details
    std::cout << "name: " + _name << std::endl;
    std::cout << "amount: " + std::to_string(outstanding) << std::endl;
}

void printBanner()
{
    std::cout << "*************************" << std::endl;
    std::cout << "***** Customer Owes *****" << std::endl;
    std::cout << "*************************" << std::endl;
}
```

### 范例二：有局部变量

```c++
void printOwing()
{
    printBanner();

    // calculate outstanding
    auto outstanding = 0.f;
    for (auto each : _orders)
    {
        outstanding += each.getAmount();
    }

    // print details
    std::cout << "name: " + _name << std::endl;
    std::cout << "amount: " + std::to_string(outstanding) << std::endl;
}
```

提炼“print details”为一个带参数的函数：

```c++
void printOwing()
{
    printBanner();

    // calculate outstanding
    auto outstanding = 0.f;
    for (auto each : _orders)
    {
        outstanding += each.getAmount();
    }

    printDetails(outstanding);
}

void printOutstanding(float outstanding)
{
    std::cout << "name: " + _name << std::endl;
    std::cout << "amount: " + std::to_string(outstanding) << std::endl;
}
```

### 范例三：对局部变量再赋值

```c++
void printOwing()
{
    printBanner();

    // calculate outstanding
    auto outstanding = 0.f;
    for (auto each : _orders)
    {
        outstanding += each.getAmount();
    }

    printDetails(outstanding);
}
```

提炼“calculate outstanding”部分代码：

```c++
void printOwing()
{
    printBanner();
    auto outstanding = getOutstanding();
    printDetails(outstanding);
}

float getOutstanding()
{
    auto result = 0.f;
    for (auto each : _orders)
    {
        result += each.getAmount();
    }
    return result;
}
```