# 类

## 类占用的内存大小决定因素

1.非静态成员变量的大小
```cpp
#include <iostream>
using namespace std;
class A {    
   static int a; // 4字节    
   int c;   // 4字节    
   int i;    // 4字节
   };
int main() {   
  cout << sizeof(A) << endl;  // 输出8
 }
```
解析：

int c(4字节) + int i(4字节) = 8字节。

二、内存对齐（Alignment）和填充（Padding）
```cpp
#include <iostream>
using namespace std;
class B {    
     char c;     // 1字节    
     double d;   // 8字节
};
 int main() {   
    cout << sizeof(B) << endl;  // 输出16
}
```
解析：

为8字节对齐，结构为：

char c(1字节) + 填充7字节 + double d(8字节) = 16字节。

三、虚函数的影响（虚函数表指针）
```cpp
#include <iostream>
using namespace std;
class C {    
    virtual void f() {}  // 引入vptr指针 8字节    
    int x; // 4 + 4
};
int main() {    
cout << sizeof(C) << endl;  // 输出16
}
```
解析：

虚函数表指针(vptr)：8字节

int x：4字节 + 4字节填充 = 16字节。

四、虚继承的影响（虚基类指针）
```cpp
#include <iostream>
using namespace std;
class Base {
    int a;
};
class Derived : virtual public Base {    
     int b;
};
int main() {    
    cout << sizeof(Derived) << endl;  // 输出24
}
```
解析：

虚继承加入虚基类指针(vbptr)：8字节

自身成员int b(4字节) + 填充4字节 = 16字节。

五、空类大小为1的原因
```cpp
#include <iostream>
using namespace std;
class Empty {};
int main() {    
     cout << sizeof(Empty) << endl;  // 输出1
}
```
解析（重点）：

空类大小不是0字节，而是至少1字节。

因为C++标准要求每个对象都有独立、唯一的内存地址，若为空类大小为0字节，多个空类实例就无法拥有不同地址，因此至少分配1字节空间以确保地址独立性。

【总结表格】

|因素	|影响类大小情况|
|---|---|
|非静态成员变量	|按类型大小累计|
|内存对齐（Padding）|	补齐字节以满足对齐要求|
|虚函数	|产生虚函数表指针（vptr）|
|虚继承|	产生虚基类指针|
|空类|	最小为1字节，确保地址独立|


## C++ 中一个空类（没有成员变量和函数），默认有哪几个函数，请写出函数定义?

编译器默认生成的函数

|函数|	是否默认生成	|用途说明|
|---|---|---|
|默认构造函数|	✅	|Empty()，用于默认构造对象|
|析构函数|	✅	|~Empty()，用于销毁对象|
|拷贝构造函数|	✅	|Empty(const Empty&)，用于复制对象|
|拷贝赋值运算符|	✅	|Empty& operator=(const Empty&)，用于赋值|
|移动构造函数（C++11 起）|	✅	|Empty(Empty&&)，用于移动语义|
|移动赋值运算符（C++11 起）|	✅|	Empty& operator=(Empty&&)，用于移动赋值|

展开写法（模拟编译器默认生成）
```cpp
class Empty {
public:
    // 默认构造函数
    Empty() = default;

    // 析构函数
    ~Empty() = default;

    // 拷贝构造函数
    Empty(const Empty& other) = default; 

    // 拷贝赋值运算符
    Empty& operator=(const Empty& other) = default;

    // 移动构造函数
    Empty(Empty&& other) = default;

    // 移动赋值运算符
    Empty& operator=(Empty&& other) = default;
};

#include <iostream>
#include <utility>
class Empty{};
int main() {
    Empty e1;
    Empty e2 = e1;
    e2 = e1;


    Empty* p = new Empty(); 
    delete p;

    Empty e3 = std::move(e1);
    e3 = std::move(e2);
    return 0;
}
```
总结：  
    空类默认有 6 个特殊函数（构造、析构、拷贝/移动构造、拷贝/移动赋值），除非你显示禁用或重写，否则编译器会默认生成它们（符合 Rule of Zero）。