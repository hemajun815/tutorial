# String相关函数一览
## 说明
C语言中，使用有关字符串处理的库函数，务必包含头文件string.h，即`#include<string.h>`。
## 预览
* strtok()：字符串分割函数。
* strstr()：字符串查找函数。
* strspn()：字符查找函数。
* strrchr()：定位字符串中最后出现的指定字符。
* strpbrk()：定位字符串中第一个出现的指定字符。
* strncpy()：复制字符串。
* strncat()：字符串连接函数。
* strncasecmp()：字符串比较函数(忽略大小写)。
* strlen()：字符串长度计算函数。
* strdup()：复制字符串。
* strcspn()：查找字符串。
* strcpy()：复制字符串。
* strcoll()：字符串比较函数(按字符排列次序)。
* strcmp()：字符串比较函数(比较字符串)。
* strchr()：字符串查找函数(返回首次出现字符的位置)。
* strcat()：连接字符串。
* strcasecmp()：字符串比较函数(忽略大小写比较字符串)。
* rindex()：字符串查找函数(返回最后一次出现的位置)。
* index()：字符串查找函数(返回首次出现的位置)。
* toupper()：字符串转换函数(小写转大写)。
* tolower()：字符串转换函数(大写转小写)。
* toascii()：将整数转换成合法的ASCII码字符。
* strtoul()：将字符串转换成无符号长整型数。
* strtol()：将字符串转换成长整型数。
* strtod()：将字符串转换成浮点数。
* gcvt()：将浮点型数转换为字符串(四舍五入)。
* atol()：将字符串转换成长整型数。
* atoi()：将字符串转换成整型数。
* atof()：将字符串转换成浮点型数。
## 详解
#### 比较字符串大小函数
1. 忽略大小写---strcasecmp
    * 函数原型：`int strcasecmp (const char *s1, const char *s2);`
    * 函数说明：用来比较参数s1和s2字符串，比较时会自动忽略大小写的差异。
    * 返回值：若参数1中字符串和参数中2字符串相同则返回0；若参数1中字符串长度大于参数2中字符串长度则返回大于0 的值；若参数1中字符串 长度小于参数2中字符串 长度则返回小于0的值。
2. 忽略大小写—stricmp
    * 函数原型：`int stricmp(char *str1, char *str2);`
    * 函数说明：以大小写不敏感方式比较两个串。
    * 返回值：若参数1中字符串和参数中2字符串相同则返回0；若参数1中字符串长度大于参数2中字符串长度则返回大于0 的值；若参数1中字符串 长度小于参数2中字符串 长度则返回小于0的值。
3. 不忽略大小写—strcmp
    * 函数原型：`int strcmp(char*str1,char*str2);`
    * 函数说明：通过比较字串中各个字符的ASCII码，来比较参数Str1和Str2字符串，比较时考虑字符的大小写。
    * 返回值：若参数1中字符串和参数中2字符串相同则返回0；若参数1中字符串长度大于参数2中字符串长度则返回大于0 的值；若参数1中字符串 长度小于参数2中字符串 长度则返回小于0的值。
4. 比较一部分—strncmpi
    * 函数原型：`int strncmpi(char *str1, char *str2, unsigned maxlen);`
    * 函数说明：比较字符串str1和str2的前maxlen个字符。
    * 返回值：若参数1中字符串和参数中2字符串相同则返回0；若参数1中字符串长度大于参数2中字符串长度则返回大于0 的值；若参数1中字符串 长度小于参数2中字符串 长度则返回小于0的值。
5. 内存区域比较—memcmp
    * 函数原型：`int memcmp(void*buf1,void *buf2,unsigned int count);`
    * 函数说明：比较内存区域buf1和buf2的前count个字节。
    * 返回值：若参数1中字符串和参数中2字符串相同则返回0；若参数1中字符串长度大于参数2中字符串长度则返回大于0 的值；若参数1中字符串 长度小于参数2中字符串 长度则返回小于0的值。
6. 内存区域部分比较—memicmp
    * 函数原型：`int memicmp(void*buf1,void*buf2,unsigned int count);`
    * 函数说明：比较内存区域buf1和buf2的前count个字节，但不区分大小写。
    * 返回值：若参数1中字符串和参数中2字符串相同则返回0；若参数1中字符串长度大于参数2中字符串长度则返回大于0 的值；若参数1中字符串 长度小于参数2中字符串 长度则返回小于0的值。
#### 从字符串中提取子串
1. 提取子串—strstr
    * 函数原型：`char* strstr(char*src,char*find);`
    * 函数说明：从字符串src中寻找find第一次出现的位置（不比较结束符NULL）。
    * 返回值：返回指向第一次出现find位置的指针，如果没有找到则返回NULL。
2. 提取分隔符间字串—strtok
    * 函数原型：`char *strtok(char*src， char*delim);`
    * 函数说明：分解字符串为一组字符串，src为要分解的字符串，delim为分隔符字符串。
    * 返回值：从src开头开始的一个个被分割的串。当没有被分割的串时则返回NULL。
#### 字符串复制
1. 字串复制—strcpy
    * 函数原型：`char*strcpy(char*dest,char*src);`
    * 函数说明：把src所指由NULL结束的字符串复制到dest所指的数组中。
    * 返回值：返回指向dest的指针。
2. 字串复制—strdup
    * 函数原型：`char* strdup(char*src);`
    * 函数说明：复制字符串src。
    * 返回值：返回指向被复制字符串的指针，所需空间有`malloc()`分配且由`free()`释放。
3. 内存空间复制—memcpy
    * 函数原型：`void *memcpy(void *dest,void *src,unsigned int count);`
    * 函数说明：src和dest 所指内存区域不能重叠；由src所致内存区域复制count个字节到dest所指内存区域中。
    * 返回值：返回指向dest的指针。
#### 字符串连接
1. 接尾连接—strcat
    * 函数原型：`char* strcat(char*dest,char*src);`
    * 函数说明：把src所指字符串添加到dest结尾处(覆盖dest结尾处的'\0')并添加'\0'。
2. 部分连接—strncat
    * 函数原型：`char* strncat(char*dest,char*src,int n);`
    * 函数说明：把src所指字符串的前n个字符添加到dest结尾处（覆盖dest结尾处的’\0’）并添加’’\0’。
    * 返回值：返回指向dest的指针。
#### 字符串查找
1. 内存区域找字符—memchr
   * 函数原型：`void *memchr(void*buf,char ch,usigned count)`
   * 函数说明：从buf所指内存区域的前count个字节查找字符ch，当第一次遇到字符ch时停止查找。
   * 返回值：如果找到了，返回指向字符ch的指针；否则返回NULL。
2. 字串中找字符—strchr
   * 函数原型：`char* strchr(char*src,char ch);`
   * 函数说明：查找字符串s中首次出现字符ch的位置。
   * 返回值：返回首次出现c的位置的指针，如果s中不存在c则返回NULL。
3. 搜索出现的字符—strcspn
   * 函数原型：`int strcspn(char*src,char*find);`
   * 函数说明：在字符串src中搜寻find中所出现的字符。
   * 返回值：返回第一个出现的字符在src中的下标值，即src中出现而不在find中出现的字串的长度。
4. 匹配任一字符—strpbrk
   * 函数原型：`char*strpbrk(char*s1,char*s2)`
   * 函数说明：在字符串S1中寻找字符串S2中任何一个字符相匹配的第一个字符的位置，空字符不包括在内。
   * 返回值：返回指向S1中第一个相匹配的字符的指针，如果没有匹配字符则返回空指针。
#### 其他函数
1. 全部转成大写—strupr
   * 函数原型：`char*strupr(char*src);`
   * 函数说明：将字符串src转换成大写形式，只转换src中出现的小写字母，不改变其他字符。
   * 返回值：返回指向src的指针。
2. 全部转成小写—strlwr
   * 函数原型：`char*strlwr(char*src);`
   * 函数说明：将字符串src转换成小写形式，只转换src中出现的大写字母，不改变其他字符。
   * 返回值：返回指向src的指针。
3. 将字串逆向—strrev
   * 函数原型：`char*strrev(char*src);`
   * 函数说明：把字符串src的所有字符的顺序颠倒过来（不包括NULL）。
   * 返回值：返回指向颠倒顺序后的字符串指针。
