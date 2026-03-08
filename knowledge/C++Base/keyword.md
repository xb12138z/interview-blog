# 关键字

## mutable关键字

1.mutable 关键字的作用

mutable 关键字用于 允许在 const 成员函数或 const 对象中修改该成员变量。适用于类的成员变量，通常用于 缓存、延迟计算、日志记录等场景。

2.mutable 的使用场景

在 const 成员函数中修改变量，与 std::thread 结合，修改 lambda 捕获的变量

用于缓存计算结果，提高程序性能 用于调试或日志记录，不影响对象的逻辑常量性

3.mutable 示例代码

1：const 成员函数修改变量
```cpp
#include <iostream>
class Test {
private:
    mutable int counter = 0; // 允许修改
public:
    void show() const {
        counter++; // `const` 函数内可修改 `mutable` 变量
        std::cout << "Counter: " << counter << std::endl;
    }
};
int main() {
    Test t;
    t.show();
    t.show();
    return 0;
}
```
输出：

Counter: 1

Counter: 2

2：用于 Lambda 捕获
```cpp
#include <iostream>
#include <thread>
#include <functional>

int main() {
    int count = 0; 

    auto func = [count]() mutable { // 在 lambda 捕获变量时加 `mutable`
        count++; // 允许修改 `count`
        std::cout << "Count in lambda: " << count << std::endl;
        };

    func();
    func();

    std::cout << count << std::endl;

    return 0;
}
```
输出：

Count in lambda: 1

Count in lambda: 2

4.面试常见问题
✅ Q1: mutable 关键字的作用是什么？

允许 const 成员函数或 const 对象修改该变量。用于修饰lambda表达式

✅ Q2: mutable 适用于哪些场景？

缓存（Cache）

日志记录（Logging）

懒加载（Lazy Evaluation）

✅ Q3: mutable 是否能用于静态成员变量？

不能，因为 static 变量是全局共享的，不受 const 限制。

✅ Q4: mutable 能否用于普通变量？

不能，mutable 只能修饰类的成员变量。

5.mutable 适用场景总结

|场景|	说明|
|---|---|
|const 成员函数|	允许在 const 成员函数中修改变量，如计数器、日志等|
|Lambda 表达式|	允许 mutable 捕获变量在 lambda 内修改|
|缓存优化|	存储计算结果，避免重复计算，提高性能|



## explicit 关键字
一、explicit 是什么？

explicit 是 C++ 提供的一个关键字，用于修饰构造函数或类型转换函数

目的：禁止隐式类型转换

二、为什么需要 explicit？

默认情况下，单参数构造函数允许隐式转换：
```cpp
class A {
public:
    A(int x) { }
};

void foo(A a) {}

foo(10); //  合法，int 隐式转换为 A
```
这可能带来意料之外的行为，导致 bug 难以发现。

三、使用 explicit 后的效果
```cpp
class A {
public:
    explicit A(int x) { }
};

foo(10);      //  错误：禁止隐式转换
foo(A(10));   //  正确：显式构造
```

四、哪些情况下应加 explicit？

|场景|	是否加 explicit|
|---|---|
|单参数构造函数	| 必须加|
|具有默认参数的单参数构造函数	| 建议加|
|类型转换构造函数（如 string）	| 建议加|
|多参数构造函数	 |不需要（无法隐式转换）|

五、explicit 还能修饰什么？

C++11 开始，支持修饰类型转换函数：
```cpp
class A {
public:
    explicit operator bool() const { return true; }
};

A a;
if (a) { }         //  合法
int x = a;         //  不允许隐式转换为 int
```
六、总结

explicit 防止隐式类型转换，避免潜在 bug，是构造函数的保护盾！

七、面试延伸问题 + 回答案例

Q1：为什么 STL 容器几乎所有构造函数都加了 explicit？

回答：

因为 STL 容器如 vector(int n)、map(Compare comp) 等都有单参数构造，如果不加 explicit，就容易在赋值、初始化时发生隐式类型转换，可能引发逻辑错误。加上 explicit 可强制开发者使用显式方式初始化容器，提升安全性和可读性。

Q2：默认构造函数加 explicit 有什么影响？

回答：

对默认构造函数使用 explicit 意味着禁止像 T obj = {}; 这样的隐式初始化，必须使用 T obj{}; 或 T obj(); 显式方式。这在某些框架中可防止误初始化，但一般不推荐对默认构造函数加 explicit，除非有特殊限制要求。

Q3：explicit 与 delete 联用有什么作用？

回答：

二者可联合用于限制特定构造方式。例如可以 delete 某个隐式转换构造，再对另一个构造函数加 explicit，从而更精细控制对象构造规则。

八、面试推荐回答模板（30 秒）
“explicit 是为了防止单参数构造函数发生隐式类型转换，容易造成逻辑错误。它要求必须显式调用构造函数或转换函数。特别适用于资源类、容器类、字符串封装等，防止出现奇怪的自动转换行为。”

## const 和 constexpr 
1.const 常量

定义：const 用于定义常量，在程序中声明后其值无法修改。常量的值在运行时确定，不能保证在编译期求值。

语法：

const int a = 10;

特点：

const 常量的值是不可修改的。

它的值在编译时无法保证是常量。

const 可以用于常量指针、常量成员函数等。

示例：
```cpp
void test() {
    const int a = 1;
    int b = a + 1;  // 运行时计算
}
```
汇编代码：
```cpp
test():
    push    rbp
    mov     rbp, rsp
    mov     DWORD PTR [rbp-4], 1     # a = 1
    mov     eax, DWORD PTR [rbp-4]   # eax = a
    add     eax, 1                   # eax += 1
    mov     DWORD PTR [rbp-8], eax   # b = eax
    nop
    pop     rbp
    ret
//结果：b 的计算在运行时完成，存在计算过程。
```

2.constexpr 常量
定义：constexpr 用于定义编译期常量，值必须在编译期计算并确定。适用于常量表达式。

语法：

constexpr int a = 10;

特点：

constexpr 常量的值必须在编译时求得。

constexpr 只能用于具有常量表达式的函数或变量。

constexpr 函数要求返回值是编译期常量。

示例：
```cpp
void test() {
    constexpr int a = 1;
    int b = a + 1;  // 编译时计算
}
```
汇编代码：
```cpp
test():
    push    rbp
    mov     rbp, rsp
    mov     DWORD PTR [rbp-4], 1      # a = 1
    mov     DWORD PTR [rbp-8], 2      # b = 2 (直接用立即数)
    nop
    pop     rbp
    ret
//结果：b 的计算在编译时完成，减少了计算过程，汇编指令显著减少。
```

3.const 与 constexpr 对比

|特性|	const|	constexpr|
|编译期计算	|不保证|	必须保证编译期计算|
|用途|	运行时常量、常量指针等|	编译期常量、编译期函数等|
|是否能用于常量表达式	|不一定	|必须是常量表达式|
|性能|	较低，需运行时计算|	高效，计算结果直接嵌入到编译后的代码中|

4.constexpr 无法使用的情况

constexpr 有一定的限制，并非在所有情况下都能使用。以下是一些不能使用 constexpr 的情况：

1.返回值不是常量表达式

constexpr 函数的返回值必须是编译期常量。如果函数的返回值不是常量表达式，则不能将其定义为 constexpr。

示例：
```cpp
int getValue() {
    return 5;  // 运行时返回
}

constexpr int square() {
    return getValue() * getValue();  // 错误：getValue() 不是常量表达式
}
```
编译错误：getValue() 不是常量表达式，因此无法用于 constexpr 函数中。

2.动态内存分配

constexpr函数不能包含动态内存分配（如new和delete），因为这些操作需要在运行时进行。

示例：
```cpp
constexpr int* allocateMemory() {
    return new int(10);  // 错误：不能在 constexpr 中使用 new
}
```
编译错误：constexpr 函数中不允许使用动态内存分配。

3.依赖于运行时输入的表达式
constexpr 只能处理编译期可确定的表达式，不能依赖于运行时输入。

示例：
```cpp
constexpr int addUserInput(int x) {
    int input;
    std::cin >> input;  // 错误：不能从输入流获取值
    return x + input;
}
```
编译错误：constexpr 函数无法依赖于运行时输入。

5.总结

const用于定义不可变的常量，但其值不一定在编译时已知，适用于运行时常量。

constexpr用于定义编译期常量，并且可以用于constexpr函数，这要求其值必须在编译时计算。

constexpr能显著提高程序的性能，尤其在需要频繁访问常量的场合。

练习题

将以下代码中的 const 常量改为 constexpr 常量，并观察输出汇编的变化。

尝试编写一个 constexpr 函数，确保其返回值在编译期求得。

调整一个包含动态内存分配的函数，使其能符合constexpr的要求。