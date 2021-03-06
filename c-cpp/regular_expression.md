# 正则表达式

## 概述

正则表达式，又称规则表达式，英文名为 Regular Expression， 在代码中常简写为 regex， regexp 或 RE， 是计算机科学的一个概念。正则表通常被用来检索、替换那些符合某个模式 (规则) 的文本。

正则表达式是对字符串 (包括普通字符，例如：a 到 z 之间的字母) 和特殊字符 (称为 “元字符” ) 操作的一种逻辑公式，就是用事先定义好的一些特定字符，及这些特定字符的组合，组成一个“规则字符串”， 这个“规则字符串”用来表达对字符串的一种过滤逻辑。正则表达式是一种文本模式，模式描述在搜索文本时要匹配的一个或多个字符串。

## C++ 的支持

C++ 从 std=c++11 之后支持正则表达式的使用，使用时只需导入相应的头文件即可，如下：

```c++
#include <regex>
using namespace std;
```

## 常用函数

### regex_match

`regex_match` 可以判断整个字符串是否匹配模式。使用示例如下：

```c++
string str = "0,1 0,2;0,0 1,0;0,1 1,1;0,2 1,2;1,0 1,1;1,1 1,2;1,1 2,1;1,2 2,2;2,0 2,1";
regex reg1("^(\\d,\\d \\d,\\d;?)+$");
smatch sma;
cout << boolalpha << regex_match(str, sma, reg1) << endl; // true
cout << sma.str() << endl; // 0,1 0,2;0,0 1,0;0,1 1,1;0,2 1,2;1,0 1,1;1,1 1,2;1,1 2,1;1,2 2,2;2,0 2,1
```

### sregex_iterator

`sregex_iterator` 可以返回多个匹配的结果。使用示例如下：

```c++
string str = "0,1 0,2;0,0 1,0;0,1 1,1;0,2 1,2;1,0 1,1;1,1 1,2;1,1 2,1;1,2 2,2;2,0 2,1";
regex reg2("\\d,\\d \\d,\\d");
sregex_iterator it(str.begin(), str.end(), reg2);
sregex_iterator end;
// 循环输出所有满足条件的结果。
while (it != end)
{
    cout << it->str() << endl;
    it++;
}
```

### regex_search

`regex_search` 可以返回第一个匹配的结果。使用示例如下：

```c++
string str = "0,1 0,2;0,0 1,0;0,1 1,1;0,2 1,2;1,0 1,1;1,1 1,2;1,1 2,1;1,2 2,2;2,0 2,1";
regex reg3("\\d,\\d \\d,\\d");
cout << boolalpha << regex_search(str, sma, reg3) << endl; // true
cout << sma.str() << endl; // 0,1 0,2
```
