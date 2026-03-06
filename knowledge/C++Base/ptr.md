# 智能指针

## C++ 主要有 3 种智能指针（位于 <memory> 头文件中）：  

1.std::unique_ptr<T>（独占所有权）。不能被复制，只能移动（std::move）。适用于独占资源管理（如文件、网络连接）。

用 std::make_unique<T>(args...) 创建（C++14+）。  

2.std::shared_ptr<T>（共享所有权）  采用引用计数，多个 shared_ptr 可共享同一对象，最后一个销毁时释放资源。 存在循环引用风险，可配合 std::weak_ptr 解决。  

用 std::make_shared<T>(args...) 创建，减少内存分配开销。  

3.std::weak_ptr<T>（弱引用）依赖 shared_ptr，不会增加引用计数。用于解决 shared_ptr 循环引用问题。可通过 lock() 获取 shared_ptr，判断对象是否仍然有效。  

## std::shared_ptr<T> 原理是什么?，请手动实现

原理分析

每个 shared_ptr 实例指向同一对象的其他 shared_ptr 实例共享一个计数器。当创建新 shared_ptr 或拷贝现有 shared_ptr 时，计数器增加。

当 shared_ptr 被销毁（例如通过析构函数）或重置（通过 reset 方法）时，计数器减少。当计数器为零时，对象被自动删除，通常通过调用 delete 操作符。

这种机制确保了对象在最后一个 shared_ptr 销毁时被释放，防止了内存泄漏。特别适合多线程或复杂数据结构中需要共享对象的场景。

手写 shared_ptr 实现 , 以下是手写 shared_ptr 的简化实现，适合面试场景：
```cpp
#include <iostream>
#include <stdexcept>

// 辅助类：用于管理共享指针的引用计数
template<typename T>
class SharedCount {
private:
    T* ptr;   // 指向管理的对象
    int count; // 共享指针的引用计数

    // 禁止拷贝构造和拷贝赋值，避免多次管理同一个资源
    SharedCount(const SharedCount&) = delete;
    SharedCount& operator=(const SharedCount&) = delete;

public:
    // 构造函数，初始化资源指针并设置引用计数为1
    SharedCount(T* p) : ptr(p), count(1) {}

    // 析构函数，释放资源
    ~SharedCount() {
        delete ptr;
    }

    // 增加引用计数
    void increment() {
        count++;
    }

    // 减少引用计数，当引用计数归零时，释放自己
    void decrement() {
        if (--count == 0) {
            delete this;
        }
    }

    // 获取指向的对象指针
    T* get() const {
        return ptr;
    }
};

// 自定义 `shared_ptr` 智能指针类
template<typename T>
class shared_ptr {
private:
    T* ptr;                  // 指向管理的对象
    SharedCount<T>* countPtr; // 共享引用计数对象指针

public:
    // 默认构造函数，可以传入裸指针 p
    explicit shared_ptr(T* p = nullptr) : ptr(p) {
        if (p) {
            try {
                countPtr = new SharedCount<T>(p); // 创建引用计数对象
            } catch (...) {
                delete p; // 如果 `new` 失败，释放资源，避免内存泄漏
                throw;
            }
        } else {
            countPtr = nullptr;
        }
    }

    // 拷贝构造函数，实现共享所有权
    shared_ptr(const shared_ptr& other) : ptr(other.ptr), countPtr(other.countPtr) {
        if (countPtr) {
            countPtr->increment(); // 增加引用计数
        }
    }

    // 移动构造函数，转移资源所有权，原对象归零
    shared_ptr(shared_ptr&& other) noexcept : ptr(other.ptr), countPtr(other.countPtr) {
        other.ptr = nullptr;
        other.countPtr = nullptr;
    }

    // 析构函数，减少引用计数，如果为零则释放资源
    ~shared_ptr() {
        if (countPtr) {
            countPtr->decrement();
        }
    }

    // 拷贝赋值运算符
    shared_ptr& operator=(const shared_ptr& other) {
        if (this != &other) { // 避免自赋值
            if (countPtr != other.countPtr) { // 只有在不同对象之间赋值时才调整
                if (countPtr) {
                    countPtr->decrement(); // 旧的对象引用计数减少
                }
                ptr = other.ptr;
                countPtr = other.countPtr;
                if (countPtr) {
                    countPtr->increment(); // 新对象引用计数增加
                }
            }
        }
        return *this;
    }

    // 移动赋值运算符
    shared_ptr& operator=(shared_ptr&& other) noexcept {
        if (this != &other) {
            if (countPtr) {
                countPtr->decrement(); // 释放原有对象的引用
            }
            ptr = other.ptr;
            countPtr = other.countPtr;
            other.ptr = nullptr;
            other.countPtr = nullptr;
        }
        return *this;
    }

    // 重载箭头运算符，支持 `shared_ptr->成员` 访问
    T* operator->() const {
        return ptr;
    }

    // 重载解引用运算符，支持 `*shared_ptr` 访问
    T& operator*() const {
        if (!ptr) {
            throw std::runtime_error("Dereferencing null shared_ptr"); // 为空时抛出异常
        }
        return *ptr;
    }

    // 比较操作符
    bool operator==(const shared_ptr& other) const {
        return ptr == other.ptr;
    }

    bool operator!=(const shared_ptr& other) const {
        return ptr != other.ptr;
    }

    // 转换为 `bool`，用于检查指针是否有效
    explicit operator bool() const {
        return ptr != nullptr;
    }

    // 重置当前 `shared_ptr`，可以传入新的原始指针
    void reset(T* p = nullptr) {
        if (p != ptr) { // 仅当新指针不同于当前指针时才执行
            if (countPtr) {
                countPtr->decrement(); // 旧的引用计数减少
            }
            ptr = p;
            if (p) {
                countPtr = new SharedCount<T>(p); // 创建新的共享计数对象
            } else {
                countPtr = nullptr;
            }
        }
    }

    // 获取当前存储的裸指针
    T* get() const {
        return ptr;
    }
};
//使用示例
int main() {
    shared_ptr<int> ptr1(new int(10));
    shared_ptr<int> ptr2 = ptr1;
    std::cout << "ptr1: " << *ptr1 << std::endl;
    std::cout << "ptr2: " << *ptr2 << std::endl;
    ptr1.reset();
    std::cout << "After ptr1.reset()" << std::endl;
    std::cout << "ptr2: " << *ptr2 << std::endl;
    shared_ptr<int> ptr3 = std::move(ptr2);
    std::cout << "After move, ptr3: " << *ptr3 << std::endl;
    std::cout << "ptr2 is now null: " << (ptr2 ? "No" : "Yes") << std::endl;
    return 0;
}
```
实现细节
为了手写 shared_ptr，我们需要实现以下核心组件：

引用计数管理类：创建一个 SharedCount 类，负责存储对象指针和引用计数，并处理计数增减及对象销毁。构造函数初始化对象指针和计数为 1。析构函数在计数为零时删除对象。increment 方法增加计数。decrement 方法减少计数，并在计数为零时删除自身和对象。

shared_ptr 类：实现智能指针的主要功能，包括构造函数、拷贝构造函数、移动构造函数、析构函数、赋值运算符（拷贝和移动）、以及必要操作符（如 operator->、operator*）。构造函数：接受原始指针，创建 SharedCount 实例，并处理异常安全。拷贝构造函数：拷贝指针和计数器指针，增加计数。移动构造函数：转移所有权，源对象设为 null。析构函数：减少计数，确保对象在计数为零时被删除。赋值运算符：处理拷贝和移动赋值，确保引用计数正确更新，并优化相同对象的情况。操作符：提供 operator-> 和 operator* 访问对象，operator bool 检查是否为 null。

标准 shared_ptr 还有更多高级特性未实现，例如：

线程安全：标准 shared_ptr 使用原子操作（如 std::atomic）确保多线程环境下的计数操作安全。面试版未考虑多线程，可能会导致数据竞争。

数组支持：标准 shared_ptr 可通过 shared_ptr<T[]> 支持数组，但需要特殊处理。

const 和 volatile 限定：标准 shared_ptr 支持 const 指针，但简化版未实现。

不完全类型：标准 shared_ptr 可用于不完全类型，但需要确保析构时类型完整。

表格：shared_ptr 关键操作与行为
|操作	       |   影响	                  |       备注|
|构造函数	    |计数初始化为 1 	       | 若指针为 null，则计数为 0|
|拷贝构造函数	|  计数增加	               |    共享所有权|
|移动构造函数   |转移所有权，源设为 null	|     计数不变|
|析构函数	    |计数减少，若为 0 则删除	| 确保对象生命周期正确|
|赋值运算符	    |更新计数，处理自赋值	    |   优化相同对象情况|
+ 

## std::make_shared 相比 std::shared_ptr<T>(new T(args...)) 有什么好处？
使用 std::make_shared<T>(args...) 相比 std::shared_ptr<T>(new T(args...)) 主要有以下几个好处：

避免额外的内存分配  

std::make_shared 会在一次内存分配中同时分配对象本体和引用计数，而 std::shared_ptr<T>(new T(args...)) 需要两次分配（一次给 T，一次给 shared_ptr 的控制块）。这不仅减少了 malloc/free 的开销，还能提高缓存命中率。

减少异常安全问题  

std::shared_ptr<T>(new T(args...)) 是两个独立的操作，new T(args...) 可能会抛出异常，而 shared_ptr 还未成功构造，导致内存泄漏。std::make_shared 进行的是原子操作，不存在这个问题。

更高效的引用计数管理  

由于 std::make_shared 在一个内存块中存储对象和引用计数，指针访问时可以减少额外的缓存访问，提高运行效率。std::shared_ptr<T>(new T(args...)) 由于分开分配对象和控制块，会导致额外的指针间接访问。

代码更简洁  

auto ptr = std::make_shared<T>(args...) 比 auto ptr = std::shared_ptr<T>(new T(args...)) 更简短，可读性更好。