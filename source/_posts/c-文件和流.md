---
title: c++文件和流
tags: c++
categories: 编程类
description: c++文件和流的使用及技巧
date: 2022-01-20 14:09:45
updated: 2022-01-20 14:09:45
cover:
keywords:
---


## C++文件和流

### 头文件

```C++
#include <iostream>
#include <fstream>
```

| 数据类型 | 描述 |
| :------ | :-- |
| ofstream | 该数据类型表示输出文件流，用于创建文件并向文件写入信息。 |
| ifstream | 该数据类型表示输入文件流，用于从文件读取信息。 |
| fstream | 该数据类型通常表示文件流，且同时具有 ofstream 和 ifstream 两种功能，这意味着它可以创建文件，向文件写入信息，从文件读取信息。 |

```C++
void open(const char *filename, ios::openmode mode);
```

openmode

| 模式标志 |    描述 |
| :------ | :--- |
| ios::app | 追加模式。所有写入都追加到文件末尾。 |
| ios::ate | 文件打开后定位到文件末尾。 |
| ios::in | 打开文件用于读取。 |
| ios::out | 打开文件用于写入。 |
| ios::trunc | 如果该文件已经存在，其内容将在打开文件之前被截断，即把文件长度设为 0。 |

```C++
    ofstream outfile;
    outfile.open("file.dat", ios::out | ios::trunc );
```

以写入模式打开文件，并希望截断文件，以防文件已存在

```C++
    ifstream  afile;
    afile.open("file.dat", ios::out | ios::in );
```

打开一个文件用于读写

文件使用结束应该手动close()

```C++
const char* path = "D:\\syq-repos\\abc.txt";
const char* path2 = "D:\\syq-repos\\abcttt.txt";
std::ifstream file(path);

std::ofstream file2(path2,std::ios::out | std::ios::trunc);
std::stringstream ssTmpFileContent;
while (!file.eof())
{
    std::string sWord;
    file >> sWord;
    file2 << sWord;
    ssTmpFileContent << sWord;
}
file2.close();
file.close();
```  

![结果](fstream-1.png)

转换后空格和换行都没有了，因为ifstream通过 >> 从文件读取数据时会在**空格或者换行**的地方隔断

```C++
char data[1024];
int nSize = 1024;
while (nSize = file.read(data, 1024).gcount())
{       
    file2.write(data, nSize);
    //file2 << data;
    ssTmpFileContent << data;
}
file2.write(data, nSize);
file2.close();
file.close();
```

![结果](fstream-2.png)

用read的方式可以把所有的字符都读取出来

## 输入输出流

### 标准 I/O

| 对象 | 描述 | 支持重定向 |
| :--- | :--- | :--- |
| cin | cin 接收从键盘输入的数据 | 是 |
| cout | cout 向屏幕上输出数据 | 是 |
| cerr | 输出警告和错误信息给程序的使用者 | 否 |
| clog | 输出程序执行过程中的日志信息(不公开) | 否 |

***cin 输入流对象常用成员方法***

| 成员方法名 | 功能 |
| :--- | :--- |
| getline(str,n,ch) | 从输入流中接收 n-1 个字符给 str 变量，当遇到指定 ch 字符时会停止读 取，默认情况下 ch 为 '\0'。 |
| get() | 从输入流中读取一个字符，同时该字符会从输入流中消失。 |
| gcount() | 返回上次从输入流提取出的字符个数，该函数常和 get()、getline()、ignore()、peek()、read()、readsome()、putback() 和 unget() 联用。 |
| peek() | 返回输入流中的第一个字符，但并不是提取该字符。 |
| putback(c) | 将字符 c 置入输入流（缓冲区）。 |
| ignore(n,ch) | 从输入流中逐个提取字符，但提取出的字符被忽略，不被使用，直至提取出 n 个字 符，或者当前读取的字符为 ch。 |
| operator>> | 重载 >> 运算符，用于读取指定类型的数据，并返回输入流对象本身。 |

***cout 输出流对象常用成员方法***

### 输入输出重定向

在默认情况下，cin 只能接收从键盘输入的数据，cout 也只能将数据输出到屏幕上。但通过重定向，cin 可以将指定文件作为输入源，即接收文件中早已准备好的数据，同样 cout 可以将原本要输出到屏幕上的数据转而写到指定文件中。

#### freopen()函数实现重定向

头文件

``` c++
#include <stdio.h>  //freopen_s
```

``` c++
const char* path = "D:\\syq-repos\\abc.txt";
const char* path2 = "D:\\syq-repos\\abcttt.txt";

std::string str;
typedef FILE* FP;
FP f1;
FP f2;
errno_t err1 = freopen_s(&f1,path,"r",stdin);
errno_t err2 = freopen_s(&f2, path2, "w", stdout);

while (std::cin >> str)
{
    std::cout << str;
}
if (f1 != nullptr)
{
    fclose(f1);
}
if (f2 != nullptr)
{
    fclose(f2);
}
```

#### rdbuf()函数实现重定向

``` c++
const char* path = "D:\\syq-repos\\abc.txt";
const char* path2 = "D:\\syq-repos\\abcttt.txt";

std::string str;

std::ifstream fin(path);
auto bufOld = std::cin.rdbuf(fin.rdbuf());
std::ofstream fout(path2,std::ios::out| std::ios::trunc);
auto bufOld2 = std::cout.rdbuf(fout.rdbuf());

while (std::cin >> str)
{
    std::cout << str;
}

std::cin.rdbuf(bufOld);
std::cout.rdbuf(bufOld2);
fin.close();
fout.close();
```

#### 通过控制台实现重定向

>C:\Users\mengma>D:\demo.exe <in.txt >out.txt

执行后会发现，控制台没有任何输出。这是因为，我们使用了"<in.txt"对程序中的 cin 输入流做了重定向，同时还用 ">out.txt"对程序中的 cout 输出流做了重定向。

如果此时读者进入 C:\Users\mengma 目录就会发现，当前目录生成了一个 out.txt 文件，其中就存储了 demo.ext 的执行结果。

>在控制台中使用 > 或者 < 实现重定向的方式，DOS、windows、Linux 以及 UNIX 都自动识别。

### tellp()和seekp()

``` c++
streampos tellp();
```

tellp() 不需要传递任何参数，会返回一个 streampos 类型值。事实上，streampos 是 fpos 类型的别名，而 fpos 通过自动类型转换，可以直接赋值给一个整形变量（即 short、int 和 long）。也就是说，在使用此函数时，我们可以用一个整形变量来接收该函数的返回值。

>注意，当输出流缓冲区中没有任何数据时，该函数返回的整形值为 0；当指定的输出流缓冲区不支持此操作，或者操作失败时，该函数返回的整形值为 -1。

seekp() 方法用于指定下一个进入输出缓冲区的字符所在的位置。

``` c++
//指定下一个字符存储的位置
ostream& seekp (streampos pos);
//通过偏移量间接指定下一个字符的存储位置   
ostream& seekp (streamoff off, ios_base::seekdir way);
```

其中，各个参数的含义如下：

+ pos：用于接收一个正整数；

+ off：用于指定相对于 way 位置的偏移量，其本质也是接收一个整数，可以是正数（代表正偏移）或者负数（代表负偏移）；

+ way：用于指定偏移位置，即从哪里计算偏移量，它可以接收表 1 所示的 3 个值。

| 模式标志 | 描 述 |
| :--- | :--- |
| ios::beg | 从文件头开始计算偏移量 |
|ios::end | 从文件末尾开始计算偏移量 |
|ios::cur | 从当前位置开始计算偏移量 |

seekp() 方法会返回一个引用形式的 ostream 类对象

### cout成员方法格式化输出

| 成员函数 | 说明 |
| :--- | :--- |
| flags(fmtfl) | 当前格式状态全部替换为 fmtfl。注意，fmtfl 可以表示一种格式，也可以表示多种格式。 |
| precision(n) | 设置输出浮点数的精度为 n。 |
| width(w) | 指定输出宽度为 w 个字符。 |
| fill(c) | 在指定输出宽度的情况下，输出的宽度不足时用字符 c 填充（默认情况是用空格填充）。 |
| setf(fmtfl, mask) | 在当前格式的基础上，追加 fmtfl 格式，并删除 mask 格式。其中，mask 参数可以省略。 |
| unsetf(mask) | 在当前格式的基础上，删除 mask 格式。|

|标 志|作 用|
| :--- | :--- |
|ios::boolapha|把 true 和 false 输出为字符串|
|ios::left|输出数据在本域宽范围内向左对齐|
|ios::right|输出数据在本域宽范围内向右对齐|
|ios::internal|数值的符号位在域宽内左对齐，数值右对齐，中间由填充字符填充|
|ios::dec|设置整数的基数为 10|
|ios::oct|设置整数的基数为 8|
|ios::hex|设置整数的基数为 16|
|ios::showbase|强制输出整数的基数（八进制数以 0 开头，十六进制数以 0x 打头）|
|ios::showpoint|强制输出浮点数的小点和尾数 0|
|ios::uppercase|在以科学记数法格式 E 和以十六进制输出字母时以大写表示|
|ios::showpos|对正数显示“+”号|
|ios::scientific|浮点数以科学记数法格式输出|
|ios::fixed|浮点数以定点格式（小数形式）输出|
|ios::unitbuf|每次输出之后刷新所有的流|

值得一提的是，当调用 unsetf() 或者 2 个参数的 setf() 函数时，为了提高编写代码的效率，可以给 mask 参数传递如下 3 个组合格式：

+ ios::adjustfield：等价于 ios::left | ios::right | ios::internal；
+ ios::basefield：等价于 ios::dec | ios::oct | ios::hex；
+ ios::floatfield：等价于 ios::scientific | ios::fixed。

```C++
double f = 123;
//设定后续以科学计数法表示浮点数
cout.setf(ios::scientific);
cout << f << '\n';
//删除之前有关浮点表示的设定
cout.unsetf(ios::floatfield);
cout << f;
```

### 使用流操纵算子格式化输出

iomanip 头文件中定义的一些常用的格式控制符，它们都可用于格式化输出。

流操纵算子|作  用
| :--- | :--- |
*dec|以十进制形式输出整数|常用
hex|以十六进制形式输出整数|
oct|以八进制形式输出整数|
fixed|以普通小数形式输出浮点数|
scientific|以科学计数法形式输出浮点数|
left|左对齐，即在宽度不足时将填充字符添加到右边|
*right|右对齐，即在宽度不足时将填充字符添加到左边|
setbase(b)|设置输出整数时的进制，b=8、10 或 16|
setw(w)|指定输出宽度为 w 个字符，或输入字符串时读入 w 个字符。注意，该函数所起的作用是一次性的，即只影响下一次 cout 输出。
setfill(c)|在指定输出宽度的情况下，输出的宽度不足时用字符 c 填充（默认情况是用空格填充）
setprecision(n)|设置输出浮点数的精度为 n。<br/>在使用非 fixed 且非 scientific 方式输出的情况下，n 即为有效数字最多的位数，如果有效数字位数超过 n，则小数部分四舍五人，或自动变为科学计 数法输出并保留一共 n 位有效数字。<br/>在使用 fixed 方式和 scientific 方式输出的情况下，n 是小数点后面应保留的位数。
setiosflags(mask)|在当前格式状态下，追加 mask 格式，mask 参数可选择表 2 中的所有值。
resetiosflags(mask)|在当前格式状态下，删除 mask 格式，mask 参数可选择表 2 中的所有值。
boolapha|把 true 和 false 输出为字符串|不常用
*noboolalpha|把 true 和 false 输出为 0、1
showbase|输出表示数值的进制的前缀
*noshowbase|不输出表示数值的进制.的前缀
showpoint|总是输出小数点
*noshowpoint|只有当小数部分存在时才显示小数点
showpos|在非负数值中显示 +
*noshowpos|在非负数值中不显示 +
uppercase|十六进制数中使用 A~E。若输出前缀，则前缀输出 0X，科学计数法中输出 E
*nouppercase|十六进制数中使用 a~e。若输出前缀，则前缀输出 0x，科学计数法中输出 e。
internal|数值的符号（正负号）在指定宽度内左对齐，数值右对 齐，中间由填充字符填充。

例：

``` c++
 //以十六进制输出整数
cout << hex << 16 << endl;
//删除之前设定的进制格式，以默认的 10 进制输出整数
cout << resetiosflags(ios::basefield)<< 16 << endl;
double a = 123;
//以科学计数法的方式输出浮点数
cout << scientific << a << endl;
//删除之前设定的科学计数法的方法
cout << resetiosflags(ios::scientific) << a << endl;
```
