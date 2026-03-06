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
🚀 最佳实践：
```cpp
if (vec.capacity() > vec.size() * 2) {
    vec.shrink_to_fit(); // 仅在容量远大于实际使用时调用
}
```
📌 总结：clear() 只清空数据但不释放内存，shrink_to_fit() 可尝试释放多余容量。