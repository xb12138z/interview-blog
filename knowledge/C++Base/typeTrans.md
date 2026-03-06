# 左值、右值

## 完美转发

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
|---|---|
|int|	int&&	|右值 |int&&|
|int&|	int& && → int&|	左值 |int&|

核心规则：

T&& 绑定左值 时，T 会被推导为 int&，最终 std::forward<int&>(arg) 仍是左值。

T&& 绑定右值 时，T 会被推导为 int，最终 std::forward<int>(arg) 仍是右值。

5. 完美转发的应用场景
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
6. 总结

|对比项|	不使用 std::forward|	使用 std::forward|
|---|---|---|
|右值传递	|变成左值，调用左值版本	|保持右值，调用右值版本|
|左值传递	|保持左值|	保持左值|
|额外拷贝	|可能会有	|避免拷贝|
|性能	|可能较低|	更高效|

面试考点：

std::forward 和 std::move 的区别？

T&& 是万能引用（Universal Reference）还是右值引用（Rvalue Reference）？

为什么 std::forward 可以避免参数类型丢失？

如何用 std::forward 实现高效的构造函数参数传递？