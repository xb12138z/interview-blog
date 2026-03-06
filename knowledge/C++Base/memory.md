#内存

## C++内存分区详解

C++ 程序在运行时，内容通常被分为五大区域，也可以简化理解为四个分区(代码、全局、堆、栈），但更严谨地说应理解为五个分区（代码、全局、常量、堆、栈）：

1.代码区 (Code/Text Segment)
存放程序的机器指令

通常是只读的，防止被编辑

2.全局区（Global Area）
包含下列内容：

已初始化的全局/静态变量（Data段）

未初始化的全局/静态变量（BSS段）

全局常量、字符串常量（.rodata 常量区）

生命周期：从程序开始到结束

全局区（全局变量、静态变量）：地址靠近、稳定

常量区（全局 const、字符串常量）：地址接近、通常只读

3.堆区 (Heap)
使用 new 或 malloc 动态分配

手动释放：delete/free

生命周期由开发者管理

（new 分配）：地址较低，向上增长

4.栈区 (Stack)
用于局部变量、函数参数等

系统自动分配和释放

生命周期短暂，随函数调用进出

（局部变量/const）：地址一般较高，向下增长


## C++ 内存泄漏有哪些？举例说明。

C++ 内存泄漏主要指程序运行过程中分配的内存未能正确释放，导致系统资源浪费。常见的内存泄漏类型如下：

1.堆内存泄露

原因：

使用new 分配的对象或 malloc 分配的内存未被 delete 或 free 释放。

示例：

```cpp
void func() {
    int* ptr = new int(10); // 未释放，导致内存泄漏
} // 函数结束，ptr 作用域消失，无法访问，但内存仍然占用
```

2.数组泄露：

原因：

使用 new[] 分配数组但未使用 delete[] 释放。

示例：

```cpp
void func() {
    int* arr = new int[100]; 
    delete arr; // 错误！应使用 delete[]
}
```

3.智能指针循环引用导致的泄漏

原因：

shared_ptr 互相引用，导致引用计数无法归零，资源不释放。

示例：
```cpp
struct A;
struct B;
struct A {
    std::shared_ptr<B> bptr;
};
struct B {
    std::shared_ptr<A> aptr;
};

void func() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    a->bptr = b;
    b->aptr = a; // 形成循环引用，导致内存泄漏
}

//解决方案
struct B;
struct A {
    std::shared_ptr<B> bptr;
};
struct B {
    std::weak_ptr<A> aptr; // 使用 weak_ptr 避免循环引用
}
```

4.资源泄露（文件、网络、线程）

原因：

未释放系统资源（文件句柄、网络连接、线程）。

示例：
```cpp
void func() {
    FILE* file = fopen("data.txt", "r");
    if (file) {
        return; // 没有 fclose(file)，文件句柄泄漏
    }
}

//解决方案：使用 RAII 封装资源，例如 std::fstream 自动管理文件资源：
void func() {
    std::ifstream file("data.txt"); // 作用域结束时自动关闭文件
}
```

5.线程泄露

原因：

创建的线程未正确回收，导致资源未释放。

示例：
```cpp
void threadFunc() {
    while (true) {}
}

void func() {
    std::thread t(threadFunc);
    // t.join(); // 若不调用 join 或 detach，会导致线程泄漏
}
```
## 虚函数表（vtable）底层原理与内存布局

什么是虚函数表（vtable）？
虚函数表（vtable）用于实现C++的运行时多态（动态绑定）。

每个含有虚函数的类都会自动生成一个虚函数表，表中存放所有虚函数的地址。

对象中存储指向虚函数表的指针（vptr），运行时通过该指针调用对应虚函数。

```cpp
classBase {
public:
    virtualvoidfunc1() {}
    virtualvoidfunc2() {}
};

classDerived : publicBase {
public:
    voidfunc1() override {}
};
```
内存布局示意图：
```cpp
Derived对象：
[vptr] --> Derived的vtable

Derived的vtable:
| func1 | --> Derived::func1()
| func2 | --> Base::func2()
```

多重继承对vtable的影响?
多重继承时，对象内存在多个vptr，分别指向每个基类对应的vtable。
```cpp
classA { virtualvoidf() {} };
classB { virtualvoidg() {} };
classC : publicA, publicB {};
对象内存布局：

C对象:
[vptr_A] --> A类vtable
[vptr_B] --> B类vtable
存在多个vptr，访问不同基类时需调整this指针偏移。
```

虚继承对vtable的影响?
虚继承解决菱形继承中重复基类实例的问题。
```cpp
引入虚基表（vbptr），记录虚基类相对对象的偏移。

classA { virtualvoidfoo(){} };
classB : virtualpublicA {};
classC : virtualpublicA {};
classD : publicB, publicC {};
虚继承对象布局：

D对象:
[vptr] --> vtable（虚函数表）
[vbptr] --> vbtable（虚基表，记录A的偏移量）
[其他成员...]
[共享的A类实例]
```

总结
虚函数表(vtable)实现运行时多态。

单继承：单个vptr简单直接。

多继承：多个vptr，每个基类独立vtable。

虚继承：额外引入虚基表(vbptr)，避免基类重复。

理解vtable的底层原理和内存布局，是C++高级面试中的重点考察内容之一。



## 一个类既不继承也不是子类，内部也没有虚函数，那它的析构函数需要声明为虚函数吗？

1.何时需要虚析构函数？
析构函数只有在 类作为基类被继承并通过基类指针删除派生类对象 时，才需要声明为 virtual。

示例（需要虚析构函数）：
```cpp
class Base {
public:
    virtual ~Base() {} // 基类析构函数应为虚函数
};

class Derived : public Base {
public:
    ~Derived() {
        std::cout << "Derived destructor" << std::endl;
    }
};

void func() {
    Base* obj = new Derived();
    delete obj; // 若 Base 没有虚析构函数，则只调用 Base 的析构函数，不调用 Derived 的析构函数，导致内存泄漏
}
```
如果 Base 的析构函数 不是 虚函数，则 delete obj 只会调用 Base 的析构函数，派生类 Derived 的析构函数不会被调用，导致派生类资源泄漏。

解决方案：在 Base 中声明 virtual ~Base()。为什么？

2.题目给出的情况分析
该类既不继承也不是子类，意味着不会被其他类继承。

该类没有虚函数，说明不会涉及多态。

没有多态、也不会通过基类指针删除对象，因此不需要虚析构函数。

示例（不需要虚析构函数）：
```cpp
class MyClass {
public:
    ~MyClass() { std::cout << "Destructor called" << std::endl; }
};
```

3.使用虚析构函数的代价
虚函数会增加 vtable（虚函数表）和 vptr（虚函数指针），增加内存开销。

调用析构函数时，会有额外的虚表查找开销。

如果类不需要虚析构函数，多余的 virtual 关键字会降低性能。

4.适用于非虚析构函数的场景
类用于值语义（如 std::vector<MyClass>）。

类不会被继承，不需要多态特性。

构造和析构的开销较大，希望避免虚函数表带来的额外消耗。
