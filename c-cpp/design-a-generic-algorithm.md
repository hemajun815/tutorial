# 泛型算法设计

本文将从具体实例出发，一步一步完成一个泛型算法的设计。

现在有一个任务：对一个整数vector，筛选其中小于10的所有元素加入到新的vector中。对应的主函数已给出，源码如下。

```c++
#include <iostream>
#include <vector>
// Add header file here...

int main(int argc, char const *argv[])
{
    const auto elem_size = 8;
    const auto filter_val = 10;
    int ia[elem_size] = {12, 8, 43, 0, 6, 21, 3, 7};
    std::vector<int> ivec_in(ia, ia + elem_size);
    std::vector<int> ivec_out;

    // Add code here...

    for (auto elem : ivec_out)
        std::cout << elem << std::endl;
    return 0;
}
```

得到这个任务，任何使用过C++语言的程序员都能马上给出如下解决方案：

```c++
std::vector<int> filter(const std::vector<int> &vec)
{
    std::vector<int> result;
    for (auto elem : vec)
        if (elem < 10)
            result.push_back(elem);
    return result;
}
```

调用方式：

```c++
// Add code here...
ivec_out = filter(ivec_in);
```

如果需求改变为筛选出所有小于20的元素，此时，能做的是，要么建立一个新函数，要么就得修改函数使其更为通用，如下：

```c++
std::vector<int> filter(const std::vector<int> &vec, const int &filter_val);
```

调用方式：

```c++
// Add code here...
ivec_out = filter(ivec_in， filter_val);
```

如果需求继续改变，允许用户指定不同的操作（如大于、小于、不等于）。可行的解决方案是以函数调用来取代`<`运算符。加入第三个参数，指定一个函数指针，其参数列表有两个整数，返回值为`bool`：

```c++
std::vector<int> filter(const std::vector<int> &vec, const int &filter_val, bool (*pred)(int, int))
{
    std::vector<int> result;
    for (auto elem : vec)
        if (pred(elem, filter_val))
            result.push_back(elem);
    return result;
}
```

调用方式：

```c++
// Add code here...
ivec_out = filter(ivec_in, filter_val, [](int x, int y) -> bool { return x < y ? true : false; });
```

此外，还可以使用`Function Object`来完成这一需求。先给出源码实现如下：

```c++
template <typename Op>
std::vector<int> filter(const std::vector<int> &vec, const int &filter_val, const Op &op)
{
    std::vector<int> result;
    std::vector<int>::const_iterator iter = vec.begin();
    while ((iter = std::find_if(iter, vec.end(), std::bind2nd(op, filter_val))) != vec.end())
    {
        result.push_back(*iter);
        iter++;
    }
    return result;
}
```

调用方式：

```c++
// Add code here...
ivec_out = filter(ivec_in, filter_val, std::less<int>());
```

所谓`Function Object`，是某种`class`对`()`运算符进行了重载，如此一来可使`Function Object`被当成一般函数来使用。代码中的`std::less<int>()`就是一个`Function Object`，此时需要引入头文件`#include <functional>`。另外，源码中的`std::find_if()`函数作用是找到符合条件的每个元素。需要引入头文件`#include <algorithm>`。

但是，上面给出的`Function Object`却不能符合`std::find_if()`函数的需要。举例说明，`std::less<type>`期望外界传入两个值，如果第一个值小于第二个值就返回`true`。但是在上面的实例中，每个元素都应该和用户提供的一个具体数值`filter_val`进行比较。理想情况下，需要做的是将`std::less<type>`转化为一个一元运算符。而这，可通过“将其第二个参数绑定到用户指定的具体数值上”完成。而标准库中提供的`Function Object Adapter`就是完成了这一操作。实现代码中的`std::bind2nd()`就是`Function Object Adapter`之一，它是将指定值绑定到第二操作数上，与之对应的还有`std::bind1st()`。

此时，需求继续升级，用户希望消除函数与数组元素类型以及vector容器类型之间的依赖关系，以使得函数更加泛型化。

为了消除函数与元素类型之间的依赖，可将函数改为函数模版。为了消除函数与容器类型之间的依赖，可传入一对迭代器（first，last），并在参数列表中增加一个用于输出的迭代器。

```c++
template <typename InputIterator, typename OutputIterator, typename ElemType, typename Op>
void filter(InputIterator first, InputIterator last, const ElemType &filter_val, const Op &op, OutputIterator out)
{
    while ((first = std::find_if(first, last, std::bind2nd(op, filter_val))) != last)
        *out++ = *first++;
}
```

对应的调用方式也会有所改变，如下：

```c++
filter(ivec_in.begin(), ivec_in.end(), filter_val, std::less<int>(), std::back_inserter(ivec_out));
```

至此，设计的`filter()`已与元素类型无关，也与操作无关，更与容器类型无关。此时的`filter()`就是一个泛型函数了。

---

> 受益于《Essential C++》(Stanley B. Lippman)一书。