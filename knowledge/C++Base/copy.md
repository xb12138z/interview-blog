# 拷贝

## 深拷贝与浅拷贝

1.引言
在 C++ 中，深拷贝（Deep Copy）和浅拷贝（Shallow Copy）是对象拷贝时的重要概念。理解这两者的区别对于避免内存泄漏和悬垂指针至关重要。

2.拷贝的基本概念
浅拷贝（Shallow Copy）：拷贝对象的成员变量，但如果成员变量是指针，仅拷贝指针地址，而非指针指向的内容。

深拷贝（Deep Copy）：不仅拷贝对象的成员变量，还会申请新内存空间，并复制指针所指向的内容。

3.浅拷贝的实现
代码示例：
```cpp
#include <iostream>
using namespace std;

class ShallowCopy {
public:
    int* data;
    ShallowCopy(int val) {
        data = new int(val);
    }
    ~ShallowCopy() {
        delete data;
    }
};

int main() {
    ShallowCopy obj1(10);
    ShallowCopy obj2 = obj1;  // 默认拷贝构造函数，进行浅拷贝
    return 0;
}
```
问题分析：

obj2 只是拷贝了 obj1 的指针地址，而没有新申请内存。

当 obj1 和 obj2 先后析构时，delete data; 会被调用两次，导致 二次释放（double free）错误。

4.深拷贝的实现
代码示例：
```cpp
#include <iostream>
using namespace std;

class DeepCopy {
public:
    int* data;
    DeepCopy(int val) {
        data = new int(val);
    }

    // 自定义拷贝构造函数，进行深拷贝
    DeepCopy(const DeepCopy& other) {
        data = new int(*(other.data));
    }

    ~DeepCopy() {
        delete data;
    }
};

int main() {
    DeepCopy obj1(10);
    DeepCopy obj2 = obj1;  // 深拷贝，data 指针指向不同的内存
    return 0;
}
```
关键点：

自定义拷贝构造函数 DeepCopy(const DeepCopy& other)，使用 new 重新分配内存并复制数据。

obj1 和 obj2 互不影响，避免了二次释放的问题。

5.深拷贝与浅拷贝的底层实现分析
5.1 浅拷贝的底层流程
ShallowCopy obj2 = obj1;
底层执行逻辑（默认拷贝构造函数）

obj2.data = obj1.data;（拷贝指针地址）

obj1 和 obj2 共享相同的内存地址。

obj2 析构时会释放 data，obj1 访问 data 可能出现 悬垂指针（dangling pointer）。

5.2 深拷贝的底层流程
DeepCopy(const DeepCopy& other) {
    data = new int(*(other.data));
}
底层执行逻辑

new int(*(other.data)); 申请新的内存并复制数据。

obj1.data 和 obj2.data 指向不同的地址。

obj1 和 obj2 独立管理各自的内存，避免了二次释放问题。

6.赋值运算符重载
如果类中使用了动态内存，则也需要重载赋值运算符，以防止浅拷贝导致的问题。

赋值运算符重载代码：
```cpp
DeepCopy& operator=(const DeepCopy& other) {
    if (this == &other) {
        return *this;
    }
    delete data;  // 释放已有内存，防止内存泄漏
    data = new int(*(other.data));  // 深拷贝
    return *this;
}
```
关键点：

先判断 this != &other 避免自赋值。

先 delete data; 释放原内存。

重新 new int(*(other.data)); 进行深拷贝。

7.使用智能指针避免拷贝问题
C++11 提供了 std::shared_ptr 和 std::unique_ptr 以管理资源，避免手动拷贝管理。

示例：
```cpp
#include <memory>
class SmartCopy {
public:
    std::shared_ptr<int> data;
    SmartCopy(int val) : data(std::make_shared<int>(val)) {}
};
```
优势：

std::shared_ptr 自动管理内存，无需手动实现深拷贝。

适用于 资源共享 的情况。

8.总结
|对比项	|浅拷贝|	深拷贝|
|---|---|
|内存分配	|仅拷贝指针地址|	重新申请内存|
|适用场景	|对象无动态内存|	对象有动态内存|
|可能问题	|二次释放、悬垂指针|	额外开销|
|解决方案	|手动管理、智能指针|	重载拷贝构造函数与赋值运算符|
面试考点：

C++ 默认的拷贝构造函数是什么？

为什么要使用深拷贝？

赋值运算符重载时为什么要 delete 旧数据？

C++11 之后如何优化拷贝管理？

建议：

避免使用裸指针，优先使用 std::unique_ptr 或 std::shared_ptr。

如果必须使用指针，记得实现 拷贝构造函数 和 赋值运算符重载。

在面试时可以结合 移动语义（C++11） 进一步优化拷贝。