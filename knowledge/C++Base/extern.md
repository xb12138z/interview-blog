# extern、inline

## inline失效场景

1.inline 作用
提示编译器内联展开函数，减少调用开销。适用于小型、简单、频繁调用的函数。

inline 仅是建议，最终由编译器决定是否展开。

2.inline 失效场景
2.1 递归函数
递归调用次数未知，编译器无法展开。
```cpp
inline int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1); // 递归调用
}
```
2.2 函数体过大
代码膨胀，影响性能，编译器可能忽略 inline。
```cpp
inline int largeFunction(int a, int b) {
    int sum = 0;
    for (int i = 0; i < 1000; i++) {
        sum += (a * b + i);
    }
    return sum;
}
```
2.3 复杂控制流（循环、条件判断）
影响优化，编译器可能不内联。
```cpp
inline int compute(int a, int b) {
    if (a > b) {
        while (b < a) {
            b++;
        }
    } else {
        for (int i = 0; i < a; i++) {
            b += i;
        }
    }
    return b;
}
```
2.4 虚函数
依赖动态绑定，运行时才确定调用目标，无法内联。
```cpp
class Base {
public:
    virtual inline void show() { // 虚函数
        std::cout << "Base::show()" << std::endl;
    }
};
```
2.5 函数指针调用
编译器无法在编译期确定具体目标函数，无法展开。
```cpp
inline int add(int a, int b) {
    return a + b;
}
int (*funcPtr)(int, int) = add; // 通过函数指针调用
int result = funcPtr(2, 3); // 无法内联
```
2.6 取函数地址
&func 需要函数实体，编译器不会内联。
```cpp
inline int multiply(int a, int b) {
    return a * b;
}
int (*ptr)(int, int) = &multiply; // 获取地址
```
2.7 动态库中的 inline
共享库函数地址运行时确定，通常不会内联。

3. inline 适用场景
✅ 短小、简单、频繁调用的函数，如 getter/setter。❌ 递归、虚函数、大函数、复杂控制流。

4. 面试高频问题
inline 是否保证函数内联？ 否，编译器决定。

递归函数为什么不能内联？ 调用次数不确定。

虚函数能否内联？ 依赖动态绑定，不能内联。

## extern"C" 详解

1.什么是 extern "C"？
作用：
extern "C" 是 C++ 提供的关键字，用于告诉编译器按照 C 语言方式编译和链接函数或变量，以解决 C++ 名称修饰（Name Mangling） 带来的兼容性问题。

2.为什么需要 extern "C"？
C++ 名称修饰（Name Mangling）问题
C++ 支持函数重载，所以编译器会在编译时修改函数名，加入参数类型信息。C 语言不支持重载，其函数名在编译后不会被修饰。

示例（C++ 代码编译后）：
```cpp
void func(int);  // 可能变为 _Z4funci
void func(double);  // 可能变为 _Z4funcd
// dumpbin /SYMBOLS main.obj | find "func"
```
如果 C++ 直接调用 C 语言函数（没有 extern "C"），链接时找不到匹配的名称，会报 undefined reference 错误。

3.extern "C" 的使用方式
（1）C++ 调用 C 语言代码
C 语言文件（c_lib.c）：
```cpp
#include <stdio.h>

void c_function() {
    printf("This is a C function\n");
}
//C 头文件（c_lib.h）：
#ifndef C_LIB_H
#define C_LIB_H

#ifdef __cplusplus
extern "C" {
#endif

void c_function();

#ifdef __cplusplus
}
#endif

#endif
//C++ 代码（main.cpp）：
#include "c_lib.h"

int main() {
    c_function();  // 成功调用 C 语言函数
    return 0;
}
```

关键点：
头文件使用 extern "C" 避免 C++ 进行名称修饰(Name Mangling)。#ifdef __cplusplus 使得 C 语言代码也能包含此头文件，不会出错。

（2）C 语言调用 C++ 代码
C++ 代码（cpp_lib.cpp）：
```cpp
#include <iostream>

extern "C" void cpp_function() {
    std::cout << "This is a C++ function\n";
}
//C 代码（main.c）：

#include <iostream>

extern "C" void cpp_function() {
    std::cout << "This is a C++ function\n";
}
```
解析

C 语言不能直接识别 extern "C"，但 C++ 端定义函数时必须加 extern "C"，这样 C 语言才能正常链接。

4.extern "C" 适用于哪些情况？
适用于

(1)函数：extern "C" void func();

(2)多个函数： extern "C" {
    void func1();
    void func2();
}

(3)变量

(4)extern "C" int global_var;整个头文件（推荐做法）

```cpp
#ifdef __cplusplus
extern "C" {
#endif

void func();

#ifdef __cplusplus
}
#endif
```
不适用于：

(1)C++ 类（C 语言不支持类）

(2)函数重载（C 语言不支持重载）

5.extern "C" 的实际应用场景

（1）C++ 调用 C 库（如 OpenCV、FFmpeg）
extern "C" {
#include <opencv2/opencv.hpp>
}

（2）封装 C 接口，提供给 Python、Rust 等其他语言使用
extern "C" int add(int a, int b) {
    return a + b;
}

（3）跨语言混合编程（C 与 C++ 混合） 嵌入式开发   操作系统开发

6.extern "C" 相关注意事项

正确使用方式：

✔ extern "C" 只能用于 全局作用域，不能嵌套到局部变量或函数内部。✔ 用 #ifdef __cplusplus 确保头文件可在 C 和 C++ 两端兼容。✔ 适用于 函数和变量，但不能用于 类和重载函数。

常见错误：

使用 extern "C" 修饰 C++ 类  或者  使用 extern "C" 修饰重载函数


7.总结

|特性	|描述|
|---|---|
|作用	|让 C++ 代码调用 C 语言代码，避免名称修饰|
|原理	|关闭 C++ 的 Name Mangling，使 C++ 符号与 C 语言兼容|
|常见用法	|extern "C" {} 包围 C 语言函数声明|
|适用范围	|函数、变量，但 不能用于类或函数重载|
|常见应用	|调用 C 语言库（如 OpenCV、FFmpeg），C++ 代码被 C 语言调用|
|extern "C" |是 C++ 和 C 语言混合编程的关键工具，使两种语言可以互相调用，提高代码的可扩展性和兼容性。|


## C++ 重定义问题全面解析：

inline vs static + namespace + extern

一、函数重定义问题的根源

在 C++ 中，如果头文件中直接定义函数，且被多个.cpp文件包含，会导致链接时重定义错误（multiple definition）。

示例：
```cpp
// math_utils.h
int add(int a, int b) {
    return a + b;
}
```
此头文件若被多个 .cpp 包含，链接时将出现重定义错误。

二、使用 static 解决重定义
```cpp
// math_utils.h
static int add(int a, int b) {
    return a + b;
}
static 将函数限制在当前翻译单元（.cpp）中，生成独立副本。
```
解决重定义，但无法共享函数定义。

三、使用 inline 解决重定义
```cpp
// math_utils.h
inline int add(int a, int b) {
    return a + b;
}
```
inline 告诉链接器：多个翻译单元中出现相同函数定义是可以的。

链接时只保留一个函数定义。

四、结合 namespace 更进一步
 
场景一：static + 命名空间
```cpp
namespace math {
    static int add(int a, int b) {
        return a + b;
    }
}
```
每个 .cpp 中仍是独立版本。

场景二：inline + 命名空间  推荐写法
```cpp
namespace math {//namespace就像唯一标识符，允许一个类中有不同namespace包含的同名函数。函数调用时指明对应的namespace即可调用对应的函数
    inline int add(int a, int b) {
        return a + b;
    }
}
```
被多个 .cpp 文件引用时也不会报错。 常用于标准库，例如 std::chrono。

场景三：匿名命名空间等价于 static
```cpp
namespace {
    int add(int a, int b) {
        return a + b;
    }
}
```
等价于 static 函数，仅在本翻译单元可见。

五、标准写法推荐模板
```cpp
#pragma once
namespace utils {
    inline int add(int a, int b) {
        return a + b;
    }
}
```

一句话总结:
static 是“躲起来”，inline 是“统一合并”，namespace 是“分类管理”。  组合使用 inline + namespace，是现代 C++ 项目最推荐的头文件函数定义方式。

附： extern 避免重定义
extern的作用是：声明一个函数或变量在别的.cpp文件中定义，它本身不参与定义。
```cpp
// math_utils.h
extern int add(int, int);  // 声明

// math_utils.cpp
int add(int a, int b) {     // 定义
    return a + b;
}
```