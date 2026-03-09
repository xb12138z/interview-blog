# 可调用对象

## 可调用对象 Callable 有哪些？

在 C++ 中，所有能使用 () 调用的对象，统称为 可调用对象，包括：

1.普通函数

2。函数指针

3.成员函数指针

4.函数对象（仿函数）

5.lambda 表达式

6.std::function 包装的任意函数体
```cpp
void f();                        // 普通函数
int (*pf)(int);                  // 函数指针
struct Functor { int operator()(int); };  // 函数对象
auto lambda = [](int x) { return x * 2; }; // Lambda
std::function<int(int)> f;       // std::function
```

2.函数对象（Functor）

示例：最基本的函数对象
```cpp
struct Adder {
    int operator()(int a, int b) const {//重载括号
        return a + b;
    }
};

int main() {
    Adder add;
    std::cout << add(3, 4) << std::endl; // 输出 7
}
```
示例：携带状态的函数对
```cpp
struct Multiplier {
    int factor;
    Multiplier(int f) : factor(f) {}
    int operator()(int x) const { return x * factor; }
};

int main() {
    Multiplier times3(3);
    std::cout << times3(10) << std::endl; // 输出 30
}
```
函数对象本质就是：一个定义了operator()的类。

3.Lambda 表达式

基本语法
```cpp
auto add = [](int a, int b) { return a + b; };
std::cout << add(3, 4); // 输出 7

//捕获变量

int x = 10, y = 20;

auto f1 = [x, &y]() { std::cout << x << ", " << y << std::endl; };
auto f2 = [&]()       { y += x; };
auto f3 = [=]()       { return x + y; };
```
lambda 本质是编译器自动生成的类，实现了operator()，即一个函数对象。

4.std::function

std::function 是一个通用的函数封装器，可以存储任意可调用对象。
```cpp
#include <functional>
#include <iostream>

int add(int a, int b) { return a + b; }

int main() {
    std::function<int(int, int)> f;

    f = add;
    std::cout << f(3, 4) << std::endl;

    f = [](int a, int b) { return a * b; };
    std::cout << f(3, 4) << std::endl;
}
```
适用于：接口参数、回调函数、异步任务、事件系统等。

5.函数指针

函数指针是最原始的“可调用对象”，可以指向普通函数。

示例：使用函数指针调用函数
```cpp
int add(int a, int b) { return a + b; }

int main() {
    int (*fp)(int, int) = &add;
    std::cout << fp(5, 7) << std::endl; // 输出 12
}
```
函数指针不能捕获上下文变量、也不支持复杂对象封装。

6.三者对比总结

|特性|	函数对象|	Lambda 表达式|	std::function|	函数指针|
|---|---|
|写法简洁|	❌|	✅	|✅|	✅|
|可捕获变量|	✅（成员变量）|	✅（捕获列表）|	✅（通过 lambda）|	❌|
|可作为回调|	✅	|✅	|✅	|✅|
|性能|	⭐⭐⭐（最优）|	⭐⭐（接近仿函数）	|⭐（慢，有堆分配）	⭐⭐|
|类型擦除支持|	❌	|❌	|✅	|❌|
|使用灵活性|	高	|高|	极高|	低|

7.回调机制实战案例
```cpp
#include <iostream>
#include <functional>

class Button {
public:
    void setOnClick(std::function<void()> cb) {
        callback = cb;
    }

    void click() {
        if (callback) callback();
    }

private:
    std::function<void()> callback;
};

int main() {
    Button btn;
    int count = 0;

    btn.setOnClick([&]() {
        std::cout << "Clicked! count = " << ++count << std::endl;
    });

    btn.click();
    btn.click();
}
```
8.面试常考问题

lambda 和函数对象的本质区别？

lambda 是编译器生成的匿名函数对象。

std::function 为啥比 lambda 慢？

因为涉及类型擦除和堆内存开销。(类型擦除见下方)

std::function 和函数指针区别？

std::function 支持任意可调用对象、类型安全、支持捕获；函数指针只能指向普通函数。

lambda 能不能作为回调函数？

可以，但如果带有捕获，需包装成 std::function。

9.推荐使用场景

|场景|	推荐使用|
|---|---|
|高性能计算、模板泛化|	函数对象|
|局部临时代码、捕获变量|	lambda 表达式|
|跨函数/模块/线程传递|	std::function|
|最简单、固定回调接口|	函数指针|


## 类型擦除

类型擦除（Type Erasure）是一种设计模式，用来隐藏对象的具体类型，统一暴露抽象接口，提供“运行时多态”。

应用场景：

std::function：封装任意可调用对象

自定义运行时插件、事件派发系统等

目标：实现一个简版 MyFunction

支持任意函数对象、lambda、函数指针等封装与调用：
```cpp
#include <iostream>
#include <functional>

// 不同类型的可调用对象
void free_func(int a) { std::cout << "自由函数: " << a << std::endl; }
struct LambdaObj { void operator()(int a) { std::cout << "lambda: " << a << std::endl; } };

int main() {
    // 类型擦除：不同类型的可调用对象，都被擦除为std::function<void(int)>
    std::function<void(int)> f;
    
    f = free_func;   // 绑定自由函数
    f(10);           // 输出：自由函数: 10
    
    f = LambdaObj(); // 绑定仿函数对象
    f(20);           // 输出：lambda: 20
    
    f = [](int a) { std::cout << "临时lambda: " << a << std::endl; };
    f(30);           // 输出：临时lambda: 30
    
    return 0;
}
```
核心结构拆解

function可以被视为由下面三个类组合而成：抽象接口类（将所有传入函数都包装成invoke，保证函数调用的统一性）【只是抽象接口，具体实现在下一个类】。包装任意类型的桥接类（负责接收一个任意的可调用的对象，把他包装成ICallable类）。类型擦除容器类（将传入的函数利用上一个类进行完整的包装，持有包装好的类的唯一指针，之后重写（）让大家可以直接使用）

1️.抽象接口类
```cpp
template<typename R, typename... Args>//返回类型R，参数类型ARGs
struct ICallable {
    // 纯虚函数：定义“调用行为”的统一规范
    // 要求实现类必须提供：接收 Args 类型参数、返回 R 类型的 invoke 方法
    virtual R invoke(Args&&... args) = 0;
    virtual ~ICallable() {}
};
```
定义通用行为：invoke()，隐藏真实类型

invoke函数三大优势：

(1)语法统一：消除普通函数、成员函数、Lambda 等可调用对象的调用语法差异，降低编码和记忆成本。

(2)泛型友好：极大简化泛型编程，无需针对不同可调用对象写分支判断，让模板更通用、简洁。

(3)现代特性：原生支持 constexpr、完美转发，适配现代 C++ 的高性能、编译期计算需求。


2️.包装任意类型的桥接类
```cpp
template<typename T, typename R, typename... Args>//T为任意可调用对象
class CallableImpl : public ICallable<R, Args...> {
    T callable;
public:
    CallableImpl(T&& c) : callable(std::move(c)) {}
    R invoke(Args&&... args) override {
        return callable(std::forward<Args>(args)...);
    }
};
```
通过模板实现对任意可调用对象的包装

3️.类型擦除容器类 MyFunction
```cpp
template<typename R, typename... Args>
class MyFunction<R(Args...)> {
    std::unique_ptr<ICallable<R, Args...>> funcPtr;

public:
    template<typename T>
    MyFunction(T&& callable) {
        funcPtr = std::make_unique<CallableImpl<T, R, Args...>>(std::forward<T>(callable));
    }

    R operator()(Args... args) const {
        return funcPtr->invoke(std::forward<Args>(args)...);
    }
};
```
将任意类型“抹去”，封装进统一接口 使用智能指针自动释放内存

完整代码示例:
```cpp
#include <iostream>
#include <memory>

template<typename R, typename... Args>
struct ICallable {
    virtual R invoke(Args&&... args) = 0;
    virtual ~ICallable() {}
};

template<typename T, typename R, typename... Args>
class CallableImpl : public ICallable<R, Args...> {
    T callable;
public:
    CallableImpl(T&& c) : callable(std::move(c)) {}
    R invoke(Args&&... args) override {
        return callable(std::forward<Args>(args)...);
    }
};

template<typename Signature>
class MyFunction;

template<typename R, typename... Args>
class MyFunction<R(Args...)> {
    std::unique_ptr<ICallable<R, Args...>> funcPtr;
public:
    template<typename T>
    MyFunction(T&& callable) {
        funcPtr = std::make_unique<CallableImpl<T, R, Args...>>(std::forward<T>(callable));
    }

    R operator()(Args... args) const {
        return funcPtr->invoke(std::forward<Args>(args)...);
    }
};

void greet() {
    std::cout << "Hello from function\n";
}

int main() {
    MyFunction<void()> f1 = [] { std::cout << "Hello from lambda\n"; };
    MyFunction<void()> f2 = greet;
    MyFunction<int(int, int)> add = [](int a, int b) { return a + b; };

    f1();  // 输出: Hello from lambda
    f2();  // 输出: Hello from function
    std::cout << "Add: " << add(2, 3) << std::endl; // 输出: 5
}
```

七、总结

|关键词|	含义|
|---|---|
|类型擦除	|屏蔽类型细节，统一接口|
|接口类	|抽象基类，提供统一方法|
|模板适配器|	用模板包装不同类型|
|虚函数机制	|运行时多态关键|
|智能指针	|管理擦除对象的生命周期|
