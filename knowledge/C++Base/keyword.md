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

4. 面试常见问题
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

5. mutable 适用场景总结
|场景|	说明|
|---|---|
|const 成员函数|	允许在 const 成员函数中修改变量，如计数器、日志等|
|Lambda 表达式|	允许 mutable 捕获变量在 lambda 内修改|
|缓存优化|	存储计算结果，避免重复计算，提高性能|