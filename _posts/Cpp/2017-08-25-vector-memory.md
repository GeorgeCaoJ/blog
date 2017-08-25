---
layout: pageshow
title: vector空间配置和栈空间返回引用的困惑和解答
category: Cpp
description: 想明白了STL空间配置器和栈空间的一点迷惑
---


### STL二级空间配置器
简单介绍一下C++STL中空间配置器的设计：  
STL空间配置器分为两级，一级是直接调用系统的C函数malloc来从heap空间申请内存，二级是通过STL内部维护的内存池来分配内存。  
具体使用哪个配置器，是通过一个阈值来判断，默认的为128Bytes;大于128字节时,调用一级配置器；小于则使用二级配置器。  
这样做的最大好处就是防止内存碎片化（分为内部碎片和外部碎片，详见CSAPP的虚拟内存章节），而且每次使用malloc从堆中申请内存时，都需要在头部存放
内存的链表信息,实际申请的内存比调用malloc时需要的其实更大一点，所以过多小额内存会使堆空间的有效利用率降低(同TCP/IP的有效载荷一个性质，基本的IP头和TCP头就占用了40个字节）。

### vector内存配置器
STL容器都是默认使用STL定义的二级空间配置器，但C++也允许用户自行设计一个更适合应用场景的配置器来替换。
但大多数情况下，默认的二级空间配置器满足使用的要求。  
这里我的疑问是，既然vector使用的是二级空间配置器，那么容器中的数据都是存放在了堆空间。如果在栈空间声明了vector，向其中添加了一个数据，能否返回其引用，
该引用保留了这个数据。  
显然，这个问题答案是不可能的，具体还是实验一下。
```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int>& func(){
    vector<int> vec{1};
    return vec;
}

int main(){
    vector<int> vec = func();
    for(auto i: vec)
        cout << i << " ";
    cout << endl;
}
```
编译时编译器就会提出警告`warning: reference to local variable 'vec' returned`，运行后直接报错`segmentation fault (core dumped)`，显然内存访问错误。
显然，在栈空间初始化的变量离开作用域的栈空间时，会默认调用析构函数，将vector的空间释放（包括vector内部定义的start, finish和end_of_storage来指示内部空间
的变量都会被析构，容器占用的空间归还给堆空间），所以遵守了C++的约定，定义在栈空间的数据不能在栈空间外访问，及时能访问也是不正确的，因为此栈空间随时可能
被后面的函数调用覆盖，另作他用。  

### 测试栈空间自动调用析构函数
```cpp
#include <iostream>
using namespace std;

struct TestCase{
    TestCase(){cout << "Constructor Called" << endl;};
    ~TestCase(){cout << "Deconstructor Called"<< endl;};
};

TestCase& test(){
    TestCase t;
    return t;
}

int main(){
    TestCase t = test();
}
```
编译运行结果是：
```
Constructor Called
Deconstructor Called
```
验证了想法。
