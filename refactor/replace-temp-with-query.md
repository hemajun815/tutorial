# 以查询取代临时变量

程序中以一个临时变量保存着某一表达式的运算结果。将这个表达式提炼到一个独立函数中。将这个临时变量的所有引用点替换为对新函数的调用。此后，新函数还可以被其他函数使用。

如：

```c++
float basePrice = _quantity * _itemPrice;
if (basePrice > 1000)
    return basePrice * 0.95;
else
    return basePrice * 0.98;
```

转变为：

```c++
if (basePrice() > 1000)
    return basePrice() * 0.95;
else
    return basePrice() * 0.98;

...

float basePrice()
{
    return _quantity * _itemPrice;
}
```

### 做法

1. 找到只被赋值一次的临时变量。
2. 将“对该临时变量赋值”之语句的等号右侧部分提炼到一个独立函数中。
3. 将所有对该变量的引用动作，替换为对它赋值的哪个表达式自身。
4. 编译、运行、测试。

### 范例

```c++
float getPrice()
{
    float basePrice = _quantity * _itemPrice;
    float discountFactor;
    if (basePrice > 1000) discountFactor = 0.95;
    else discountFactor = 0.98;
    return basePrice * discountFactor;
}
```

首先，替换`basePrice`：

```c++
float getPrice()
{
    float discountFactor;
    if (basePrice() > 1000) discountFactor = 0.95;
    else discountFactor = 0.98;
    return basePrice() * discountFactor;
}

float basePrice()
{
    return _quantity * _itemPrice;
}
```

然后，替换`discountFactor`：

```c++
float getPrice()
{
    return basePrice() * discountFactor();
}

float basePrice()
{
    return _quantity * _itemPrice;
}

float discountFactor()
{
    return basePrice() > 1000 ? 0.95 : 0.98;
}
```