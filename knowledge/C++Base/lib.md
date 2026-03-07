# 库文件

## C++动态库与静态库

1.C++ 程序编译过程分为三步： 预处理 ➞ 编译 ➞ 链接

链接的目的： 将多个 目标文件 (.o) 和 库文件 合成最终的 可执行文件

链接分为两种：【静态链接】 和 【动态链接】

2.两种链接的区别

静态链接 (Static Linking)
在 编译期 将 .a 库中的代码直接打包到可执行文件

不依赖外部库，可独立运行

动态链接 (Dynamic Linking)
在 运行期 通过 动态链接器 加载 .so/.dll 库

程序中只存在函数符号的引用，而非实际代码

3.使用示例：
```cpp
//静态链接
math.h

#ifndef MATH_H
#define MATH_H
int add(int a, int b);
#endif
math.cpp

#include "math.h"
int add(int a, int b) {
    return a + b;
}
main.cpp

#include <iostream>
#include "math.h"
int main() {
    std::cout << "2 + 3 = " << add(2, 3) << std::endl;
    return 0;
}
```
编译指令：

静态链接
```cpp
g++ -c math.cpp        # 编译成 math.o
ar rcs libmath.a math.o   # 打包成静态库

g++ main.cpp -L. -lmath -o app  # 链接库
```

动态链接
```cpp
g++ -fPIC -c math.cpp             # 使用 -fPIC 编译成目标文件
g++ -shared -o libmath.so math.o  # 生成动态库

g++ main.cpp -L. -lmath -o app    # 链接
export LD_LIBRARY_PATH=.          # 设置路径
./app
```
对比表格：

|项目|	静态链接|	动态链接|
|---|---|
|可执行体积	|大	|小|
|运行效率	|高|	略低|
|库依赖	|无外部依赖|	需外部 .so/.dll|
|升级维护	|需重新编译	|可单独更新|
|共享性	|差（每个程序一份）	|好（多程序共用）|

静态链接
全部打包带走，一劳永逸，但“重”

动态链接
只带地址引用，运行时“借用”，更灵活