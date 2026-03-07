# 类型转换

## 完美转发（左值、右值）

1.引言
在 C++11 之前，泛型函数在传递参数时无法保持参数的原始类型（左值或右值），导致额外的拷贝或移动操作。完美转发（Perfect Forwarding）是一种 高效传递参数 的技术，能够 保持参数的原始特性，避免额外的性能开销。

2.什么是完美转发？
完美转发 是指 在泛型模板函数中，以参数的原始形式（左值或右值）传递给目标函数，从而避免 不必要的拷贝或移动操作。

2.1 代码示例
```cpp
#include <iostream>
using namespace std;

void process(int& x) { cout << "Lvalue reference: " << x << endl; }
void process(int&& x) { cout << "Rvalue reference: " << x << endl; }

// 泛型函数，使用完美转发
template <typename T>
void forwardExample(T&& arg) {
    process(std::forward<T>(arg));  // 关键：std::forward 保持原始类型
}

int main() {
    int a = 10;
    forwardExample(a);   // 传递左值
    forwardExample(20);  // 传递右值
    return 0;
}
```
输出：

Lvalue reference: 10
Rvalue reference: 20
分析：

std::forward<T>(arg) 让 arg 保持左值或右值特性，从而 正确调用 process(int&) 或 process(int&&)。

forwardExample(a) 传递左值，std::forward<T>(a) 仍是左值。

forwardExample(20) 传递右值，std::forward<T>(20) 仍是右值。

3.如果去掉 std::forward 会怎样？
问题代码：
```cpp
template <typename T>
void forwardExample(T&& arg) {
    process(arg);  // 没有 std::forward
}
```
输出：

Lvalue reference: 10
Lvalue reference: 20  // 右值变成了左值！
错误分析：

arg 在 process(arg) 语境中变成了左值，即使 forwardExample(20) 传递的是右值，arg 也会 丢失右值特性。

结果：process(int&&) 无法调用，所有右值都会被当成左值，导致 额外拷贝或移动。

4.std::forward 的工作原理
std::forward<T>(arg) 通过 引用折叠（Reference Collapsing）和 类型推导 来决定参数是否应该保留右值特性。

|T 传递的类型|	T&& 推导后|	std::forward<T>(arg)| 结果|
|---|---|---|---|
|int|	int&&	|右值 |int&&|
|int&|	int& && → int&|	左值 |int&|

核心规则：

T&& 绑定左值 时，T 会被推导为 int&，最终 std::forward<int&>(arg) 仍是左值。

T&& 绑定右值 时，T 会被推导为 int，最终 std::forward<int>(arg) 仍是右值。

5.完美转发的应用场景
5.1 传递构造函数参数
```cpp
class MyClass {
public:
    template <typename T>
    MyClass(T&& arg) : data(std::forward<T>(arg)) {}
private:
    int data;
};
std::forward<T>(arg) 确保 arg 以最佳方式传递给 data，避免不必要的拷贝。
```
5.2 传递函数参数
```cpp
#include <utility>

void print(const std::string& s) {
    std::cout << "Lvalue: " << s << std::endl;
}
void print(std::string&& s) {
    std::cout << "Rvalue: " << s << std::endl;
}

// 通过完美转发调用 print
template <typename T>
void callPrint(T&& arg) {
    print(std::forward<T>(arg));
}
```
6.总结

|对比项|	不使用 std::forward|	使用 std::forward|
|---|---|---|
|右值传递	|变成左值，调用左值版本	|保持右值，调用右值版本|
|左值传递	|保持左值|	保持左值|
|额外拷贝	|可能会有	|避免拷贝|
|性能	|可能较低|	更高效|

面试考点：

1.std::forward 和 std::move 的区别？

move是无条件把值转化为右值，而forward是完美转发，输入为左值则传入左值，输入为右值则传入右值

2.T&& 是万能引用（Universal Reference）还是右值引用（Rvalue Reference）？

当T为模板是为万能引用，函数能同时接收左值和右值。这是forward的使用前提

3.为什么 std::forward 可以避免参数类型丢失？

C++中有名字的变量都会被视为左值，当你把右值（如 20）传给万能引用 T&& arg 时，T 会被推导为 int（非引用），arg 的类型是 int&&（右值引用）；但在函数内部，arg 是 “有名字的右值引用”，编译器会把它当作左值处理 —— 这就导致原始的 “右值属性” 丢失了。

4.如何用 std::forward 实现高效的构造函数参数传递？

传入右值给类的构造函数，函数中复制直接有右值赋值（移动构造）。


## C++ 类型转换及 UB（未定义行为）

C++ 中的四种类型转换运算符（static_cast、dynamic_cast、const_cast、reinterpret_cast）及其正确使用场景
1.static_cast（静态转换）
static_cast 是 C++ 提供的 编译时（静态）类型转换，主要用于已知安全的类型转换。它比 C 风格 (T)value 更加安全，具有更明确的语义，并在编译时进行类型检查.

1.1 正确用法：基本数据类型转换
```cpp
#include <iostream>
int main() {
    double d = 3.14;
    int i = static_cast<int>(d);  // 将 double 转换为 int
    std::cout << "i = " << i << std::endl;  // 输出：3
    return 0;
}
```
1.2 正确用法：基类与派生类之间的向上转换
```cpp
#include <iostream>
class Base {
public:
    virtual ~Base() {} // 为 dynamic_cast 保证多态
};

class Derived : public Base {};

int main() {
    Derived d;
    // 向上转换：从 Derived* 转为 Base*
    Base* b = static_cast<Base*>(&d);
    std::cout << "Base pointer: " << b << std::endl;
    return 0;
}
```
1.3 UB 案例：错误的向下转换
如果将一个 Base 对象错误地转换为 Derived 类型，将导致 UB。
```cpp
#include <iostream>
class Base {};
class Derived : public Base {};

int main() {
    Base b;
    // 错误：Base 对象并不包含 Derived 的信息
    Derived* d = static_cast<Derived*>(&b);  // UB：不正确的向下转换
    // 这里 d 指向的数据不包含 Derived 的特有部分，后续使用 d 可能导致崩溃或错误
    return 0;
}
```

2.dynamic_cast（动态转换）
dynamic_cast 是 C++ 运行时类型转换（RTTI, Run-Time Type Information） 机制的一部分，主要用于基类和派生类之间的转换，尤其是 安全的向下转换（Base → Derived）。

2.1 正确用法：向下转换（需要多态类型）
```cpp
#include <iostream>
class Base {
public:
    virtual ~Base() {} // 必须有虚函数
};

class Derived : public Base {
public:
    void hello() { std::cout << "Hello from Derived!" << std::endl; }
};

int main() {
    Base* b = new Derived();
    // 尝试将 Base* 动态转换为 Derived*
    Derived* d = dynamic_cast<Derived*>(b);
    if (d) {
        d->hello();  // 转换成功，调用 Derived 的方法
    } else {
        std::cout << "Conversion failed." << std::endl;
    }
    delete b;
    return 0;
}
```
2.2 UB 案例：对非多态类型使用 dynamic_cast
如果基类没有虚函数，则 dynamic_cast 无法执行正确的运行时检查，结果将是 UB。
```cpp
#include <iostream>
class Base { /* 没有虚函数 */ };
class Derived : public Base {};

int main() {
    Base* b = new Derived();
    // 尝试进行动态转换，但 Base 不是多态类型
    Derived* d = dynamic_cast<Derived*>(b);  // UB：因为 Base 没有虚函数
    if (d) {
        std::cout << "Converted successfully." << std::endl;
    } else {
        std::cout << "Conversion failed." << std::endl;
    }
    delete b;
    return 0;
}
```
3.const_cast（去除 const/volatile 限定符）

3.1 正确用法：修改可变数据
如果对象本身不是 const，通过 const_cast 去除指针的 const 属性。
```cpp
#include <iostream>
void modify(const char* str) {
    // 修改非 const 字符数组是安全的
    char* p = const_cast<char*>(str);
    p[0] = 'H';
}

int main() {
    char str[] = "hello";
    modify(str);
    std::cout << "Modified string: " << str << std::endl;  // 输出：Hello
    return 0;
}
```
3.2 UB 案例：修改真正的 const 对象
如果对象本身是 const，再通过 const_cast 修改会导致 UB。
```cpp
#include <iostream>
int main() {
    const int a = 10;
    // a 本身是 const，将其地址转换为 int* 后修改是未定义行为
    int* p = const_cast<int*>(&a);
    *p = 20;  // UB：尝试修改只读数据
    std::cout << "a = " << a << std::endl;
    return 0;
}
```
4.reinterpret_cast（重解释转换）

4.1 正确用法：指针与整数之间转换
```cpp
#include <iostream>
#include <cstdint>
int main() {
    int a = 42;
    // 将指针转换为整数
    uintptr_t addr = reinterpret_cast<uintptr_t>(&a);
    // 再转换回指针
    int* p = reinterpret_cast<int*>(addr);
    std::cout << "*p = " << *p << std::endl;  // 输出：42
    return 0;
}
```
4.2 UB 案例：错误地将不同类型的指针相互转换
如果将一个类型的指针错误地解释为另一个不兼容类型，会导致 UB。
```cpp
#include <iostream>
int main() {
    int a = 10;
    // 错误地将 int* 转换为 double*
    double* p = reinterpret_cast<double*>(&a);
    // 尝试以 double* 方式访问内存，可能破坏内存布局，导致 UB
    std::cout << "*p = " << *p << std::endl;  // UB
    return 0;
}
```
5.C 风格转换（(T)value）

5.1 案例：C 风格转换的混合行为
C 风格转换可能同时应用了 static_cast、const_cast 和 reinterpret_cast，容易引入隐患，不推荐使用。
```cpp
#include <iostream>
class Base {};
class Derived : public Base {};

int main() {
    Base b;
    // C 风格转换将 b 强制转换为 Derived*，相当于 static_cast 和 reinterpret_cast 的混合，
    // 没有运行时检查，可能导致 UB
    Derived* d = (Derived*)&b;  // UB
    return 0;
}
```