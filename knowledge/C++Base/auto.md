# 自动类型推断

## 2. auto 关键字
2.1 什么是 auto？  
auto 关键字用于自动推断变量的类型，编译器根据变量的初始值自动确定其数据类型。

示例
```cpp
#include <iostream>
#include <vector>

int main() {
    auto x = 42;        // 推断 x 为 int
    auto y = 3.14;      // 推断 y 为 double
    auto z = "hello";   // 推断 z 为 const char*

    std::cout << x << " " << y << " " << z << std::endl;
    return 0;
}
```
2.2 auto 在容器中的应用  
遍历 std::vector
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
for (auto it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " ";
}
//auto it让编译器自动推断迭代器类型，避免冗长的std::vector<int>::iterator。

//使用范围 for 循环

for (auto v : vec) {
    std::cout << v << " ";
}
```
auto 省略了显式的 int 声明，提高可读性。

2.3 auto 与 const、引用的结合  
使用 auto 推导引用
```cpp
int a = 10;
auto& ref = a;  // ref 是 int&，可以修改 a
ref = 20;
std::cout << a;  // 输出 20
 
//使用 const auto 保持常量性
const int b = 30;
auto x = b;       // x 的类型是 int，不是 const int
const auto y = b; // y 的类型是 const int
```

## decltype 关键字

3.1 decltype 的作用  
decltype 用于获取表达式的精确类型，不同于 auto，它不会立即对变量求值。

示例
```cpp
int a = 10;
decltype(a) b = 20;  // b 的类型是 int

//适用于返回值类型

template<typename T1, typename T2>
auto add(T1 a, T2 b) -> decltype(a + b) {//auto ... -> decltype(a + b)中auto为占位符，告诉程序返回值类型之后确定。而decltype(a + b)用于计算返回值类型
    return a + b;
}
```

3.2 decltype 与 auto 的区别

|关键字|	作用|	是否立即计算表达式|
|---|---|---|
|auto	|根据初始化值推导类型|	是|
|decltype	|获取表达式的类型|	否|
 
示例对比
```cpp
int a = 10, b = 20;
auto x = a + b;      // x 是 int，表达式 `a + b` 被计算
decltype(a + b) y;   // y 也是 int，但不会计算 `a + b`
```

3.3 decltype(auto)  
decltype(auto)可以保持表达式的所有修饰符（如const、引用等）

示例
```cpp
int a = 42;
int& ref = a;
decltype(auto) x = ref;  // x 是 int&，保持引用
```