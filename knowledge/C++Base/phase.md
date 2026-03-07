# C++一些语句的语法讲解

## vector语句

1.vector 扩容机制
std::vector 采用 指数增长策略，即每次扩容时 容量（capacity）约为当前的 1.5 - 2 倍。

扩容步骤：

申请更大内存（通常是当前 capacity 的 1.5 - 2 倍）。

拷贝或移动旧数据 到新内存。

释放旧内存。

示例：
```cpp
std::vector<int> vec;
for (int i = 1; i <= 10; ++i) {
    vec.push_back(i);
    std::cout << "Size: " << vec.size() << ", Capacity: " << vec.capacity() << std::endl;
}
```
🚀 优化：如果已知需要存储大量数据，可使用reserve(n)预分配空间，避免多次扩容。

vec.reserve(1000);  // 直接预分配 1000 个元素的空间

2.emplace_back() 的好处

emplace_back(args...) 原地构造对象，避免不必要的拷贝/移动。

相比 push_back(a) 更高效，特别适用于存储复杂对象。

示例：
```cpp
std::vector<std::string> vec;
vec.push_back("hello");      // 先创建 std::string，再拷贝
vec.emplace_back("world");  // 直接在 vector 内构造 std::string
```
结论：

对于简单数据类型，push_back()和emplace_back()差别不大。

对于复杂对象，优先使用 emplace_back() 以提升性能！

3.clear() 优化
clear() 仅删除元素但不会释放 capacity。

如果希望释放多余内存，可使用shrink_to_fit()，但不保证立即生效。

示例：
```cpp
std::vector<int> vec(1000);
vec.clear();  // size 变 0，但 capacity 仍然是 1000
vec.shrink_to_fit();  // 可能减少 capacity
```
最佳实践：
```cpp
if (vec.capacity() > vec.size() * 2) {
    vec.shrink_to_fit(); // 仅在容量远大于实际使用时调用
}
```
📌 总结：clear() 只清空数据但不释放内存，shrink_to_fit() 可尝试释放多余容量。


## sizeof语句和strlen语句

一、基本概念对比表格

|对比项|	sizeof|	strlen|
|---|---|
|类型|	编译期运算符（operator）|	运行时函数（#include <cstring>）|
|返回类型|	size_t（表示字节数）|	size_t（表示字符串长度）|
|参数要求|	变量 / 类型	|char*（C风格字符串）|
|是否包含 \0|	包含（数组大小）|	不包含（仅字符个数）|
|计算时间点|	编译阶段完成|	运行阶段，从头遍历直到 \0|

一般sizeof用于数组长度的计算，而strlen用于计算string长度

二、代码案例
```cpp
#include <iostream>
#include <cstring>
using namespace std;

int main() {
    char str[] = "Hello";
    char* p = str;

    cout << "sizeof(str): " << sizeof(str) << endl;   // 6（包含 \0）
    cout << "strlen(str): " << strlen(str) << endl;   // 5

    cout << "sizeof(p): " << sizeof(p) << endl;       // 8（64 位系统下指针大小）
    cout << "strlen(p): " << strlen(p) << endl;       // 5

    return 0;
}
```

三、数组在传参时退化为指针
```cpp
void test(char str[]) {
    cout << "sizeof(str) in func: " << sizeof(str) << endl; // 实际为指针大小，非数组大小
    cout << "strlen(str) in func: " << strlen(str) << endl;
}

int main() {
    char str[] = "Hello";
    test(str);  // str 会退化为 char* 类型传入函数
    return 0;
}
```

注意：  在函数参数中，char str[] 实际等价于 char* str，sizeof(str) 返回的是指针大小（如 8），而非数组大小。

四、常见误区

|写法|	错误原因|
|--
|sizeof(char*)| 得到长度	实际返回的是指针大小，不是字符串长度|
|strlen(uninit_ptr)|	空指针或未初始化指针，可能导致崩溃|
|strlen("abc\0def")	|只会计算 abc，长度为 3|
|strlen(nullptr)	|调用会导致程序崩溃|
