---
title:  【CPP】C++ 标准库（STL）系列教程 容器适配器
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 容器适配器 部分。


---
# CPP 容器适配器快速入门

### 引言

在C++标准模板库（STL）中，容器适配器是一种特殊类型的模板类，它们提供对底层容器的接口封装，从而实现了不同的数据结构操作特性。通过使用容器适配器，我们可以更加简洁、高效地操作底层容器，满足不同的编程需求。

在本文中，我们将介绍C++ STL中的几个常用容器适配器：stack、queue和priority_queue。这些容器适配器分别实现了栈、队列和优先队列的数据结构，使得我们可以方便地进行元素的入栈/出栈、入队/出队以及按照优先级顺序访问元素等操作。

接下来，我们将详细讨论这些容器适配器的使用方法、特性以及常见操作，帮助读者快速上手并使用它们。

### 容器适配器
容器适配器是类模板。其主要作用是对底层容器进行封装，仅提供特定的函数集合。

容器适配器并不直接存储数据，而是依赖于一个或多个底层容器来管理数据的存储和访问。

这意味着，当我们使用容器适配器时，实际上是在操作底层容器中的数据，而适配器为我们提供了特定的接口来执行这些操作。

或者说容器适配器保有一个底层容器作为成员对象，所有的函数都是在调用底层容器的函数。

因此如果理解我们之前讲过的 序列容器 / 关联容器 / 无序关联容器 的话，容器适配器就非常简单了。

---
### stack
std::stack 类是一种容器适配器，他实现经典数据结构中**栈**的功能 —— FILO（先进后出）。

栈的主要操作发生在被称作栈顶的容器尾部。元素通过此位置被推入（添加）或弹出（移除）。

为了使 std::stack 能够正常工作，其底层的容器必须满足序列容器的要求。此外，该容器还必须提供以下拥有标准语义的函数：
- `back()` ：类似于 `std::vector::back()`，用于访问栈顶元素。
- `push_back()` ：类似于 `std::deque::push_back()`，用于在栈顶添加元素。
- `pop_back()` ：类似于 `std::list::pop_back()`，用于从栈顶移除元素。

>标准容器 std::vector（包括 std::vector<bool>）、std::deque 和 std::list 满足这些要求。
>如果在使用 std::stack 时没有特定指定底层容器类型，那么默认情况下会使用 std::deque 作为其底层容器。
因为 std::deque 在大多数情况下的性能表现都是相当优秀的，特别是在元素的添加和移除操作方面。


#### 成员对象
容器适配器作为封装，保有一个 `Container c` 的底层容器作为受保护成员对象。
之后的各种操作函数本质上都是调用 `Container c` 的函数。
 
#### 隐式定义的成员函数
- `构造函数`: 初始化容器
- `析构函数`: 销毁容器的每个元素
>销毁 stack。调用各元素的析构函数，然后解分配所用的存储。
注意，若元素是指针，则不销毁所指向的对象。

- `operator=` : 赋值运算符
> - 复制赋值运算符。以 other 内容的副本替换内容。
    相当于调用 `c = other.c`。
> - 移动赋值运算符。用移动语义以 other 的内容替换内容（即从 other 移动 other 中的数据到此容器中）。
    相当于调用 `c = std::move(other.c)`。

#### 元素访问
- `top` : 访问栈顶元素
>返回 stack 中顶元素的引用。它是最近推入的元素。此元素将在调用 pop() 时被移除。
>实际上调用 `c.back()`。

#### 容量
- `empty` : 检查容器适配器是否为空
>检查底层容器是否为空。
实际上调用 `c.empty()`。

- `size` : 返回元素数
>返回底层容器中的元素数。
实际上调用 `c.size()`。

#### 修改器
- `push` : 向栈顶插入元素
>将给定的元素 value 推到 stack 顶。
>1) 实际上调用 `c.push_back(value)`。
>2) 实际上调用 `c.push_back(std::move(value))`。

- `emplace` : 在顶部原位构造元素
>推入新元素到栈顶。
原位构造元素，即不进行移动或复制操作。以与提供给函数严格相同的实参调用元素的构造函数。
实际上调用 `c.emplace_back(std::forward<Args>(args)...)`。

- `pop` : 移除栈顶元素
>从 stack 移除顶元素。
实际上调用 `c.pop_back()`。

- `swap` : 交换内容
>交换容器适配器与 other 的内容。
实际上调用 `swap(c, other.c)`;

- `运算符重载` : 按照字典顺序比较两个 stack 的值
>比较两个容器适配器的底层容器。通过应用对应的运算符到底层容器进行比较。
>- operator==
>- operator!=
>- operator<
>- operator<=
>- operator>
>- operator>=

- `std::swap(std::stack)` : 特化 std::swap 算法
>为 std::stack 特化了 std::swap 算法。
交换 lhs 与 rhs 的内容。
实际上调用 `lhs.swap(rhs)`。

```cpp
    // 示例
    // 创建一个空的 std::stack  
    std::stack<int> myStack;  
  
    // 向栈中添加元素  
    myStack.push(1);  
    myStack.push(2);  
    myStack.push(3);  
  
    // 检查栈是否为空  
    if (!myStack.empty()) 
    {  
        // 访问栈顶元素  
        int topElement = myStack.top();  
        std::cout << "栈顶元素是: " << topElement << std::endl;  
  
        // 移除栈顶元素  
        myStack.pop();  
  
        // 再次检查栈顶元素  
        if (!myStack.empty()) 
        {  
            topElement = myStack.top();  
            std::cout << "移除一个元素后，栈顶元素是: " << topElement << std::endl;  
        }  
    }  
  
    // 栈的大小  
    std::cout << "栈的大小是: " << myStack.size() << std::endl;  
  
    // 清空栈  
    while (!myStack.empty()) 
    {  
        myStack.pop();  
    }  
```

---
### queue
std::queue 类是一种容器适配器，他实现经典数据结构中 **队列** 的功能 ——  FIFO（先进先出）。

栈的主要操作发生在容器两端（首端和尾端）。queue 在底层容器尾端推入元素，从首端弹出元素。

为了使 std::queue 能够正常工作，其底层的容器必须满足序列容器的要求。此外，该容器还必须提供以下拥有标准语义的函数：

front()，例如 std::list::front()，
push_back()，例如 std::deque::push_back()，

- `back()` ：类似于 `std::deque::back()`，用于访问队尾元素。
- `front()` ：类似于 `std::list::front()`，用于访问队头元素。
- `push_back()` ：类似于 `std::deque::push_back()`，用于在队尾添加元素。
- `pop_front()` ：类似于 `std::list::pop_front()`，用于从队头移除元素。

>标准容器 std::deque 和 std::list 满足这些要求。
>如果在使用 std::queue 时没有特定指定底层容器类型，那么默认情况下会使用 std::deque 作为其底层容器。
因为 std::deque 在大多数情况下的性能表现都是相当优秀的，特别是在元素的添加和移除操作方面。

queue 实现思路和 stack 类模板基本一致，就是调用底层容器接口。只是操作函数的集合改变了。

#### 成员对象
容器适配器作为封装，保有一个 `Container c` 的底层容器作为受保护成员对象。
之后的各种操作函数本质上都是调用 `Container c` 的函数。
 
#### 隐式定义的成员函数
- `构造函数`: 初始化容器
- `析构函数`: 销毁容器的每个元素
>销毁 queue。调用各元素的析构函数，然后解分配所用的存储。
注意，若元素是指针，则不销毁所指向的对象。

- `operator=` : 赋值运算符
> - 复制赋值运算符。以 other 内容的副本替换内容。
    相当于调用 `c = other.c`。
> - 移动赋值运算符。用移动语义以 other 的内容替换内容（即从 other 移动 other 中的数据到此容器中）。
    相当于调用 `c = std::move(other.c)`。

#### 元素访问
- `front` : 访问第一个元素
>返回到 queue 中首元素的引用。此元素将是调用 pop() 时第一个移除的元素。
实际上调用 `c.front()`。

- `back` : 访问最后一个元素
>返回到 queue 中末元素的引用。这是最近推入的元素。
实际上调用 `c.back()`。

#### 容量
- `empty` : 检查容器适配器是否为空
>检查底层容器是否为空。
实际上调用 `c.empty()`。

- `size` : 返回元素数
>返回底层容器中的元素数。
实际上调用 `c.size()`。

#### 修改器
- `push` : 	向队列尾部插入元素
>将给定的元素 value 推到队尾。
>1) 实际上调用 `c.push_back(value)`。
>2) 实际上调用 `c.push_back(std::move(value))`。

- `emplace` : 在尾部原位构造元素
>推入新元素到队尾。
原位构造元素，即不进行移动或复制操作。以与提供给函数严格相同的实参调用元素的构造函数。
实际上调用 `c.emplace_back(std::forward<Args>(args)...)`。

- `pop` : 移除首个元素
>从 queue 移除队头元素。
实际上调用 `c.pop_front()`。

- `swap` : 交换内容
>交换容器适配器与 other 的内容。
实际上调用 `swap(c, other.c)`;

- `运算符重载` : 按照字典顺序比较两个 queue 的值
>比较两个容器适配器的底层容器。通过应用对应的运算符到底层容器进行比较。
>- operator==
>- operator!=
>- operator<
>- operator<=
>- operator>
>- operator>=

`std::swap(std::queue)` : 特化 std::swap 算法
>为 std::queue 特化了 std::swap 算法。
交换 lhs 与 rhs 的内容。
实际上调用 `lhs.swap(rhs)`。

```cpp
    // 示例
    // 创建一个空的 std::queue  
    std::queue<int> myQueue;  
  
    // 向队列中添加元素  
    myQueue.push(1);  
    myQueue.push(2);  
    myQueue.push(3);  
  
    // 检查队列是否为空  
    if (!myQueue.empty()) 
    {  
        // 访问队列的首个元素  
        int frontElement = myQueue.front();  
        std::cout << "队列的首个元素是: " << frontElement << std::endl;  
  
        // 访问队列的最后一个元素  
        int backElement = myQueue.back();  
        std::cout << "队列的最后一个元素是: " << backElement << std::endl;  
  
        // 移除队列的首个元素  
        myQueue.pop();  
  
        // 再次检查队列的首个元素  
        if (!myQueue.empty()) 
        {  
            frontElement = myQueue.front();  
            std::cout << "移除一个元素后，队列的首个元素是: " << frontElement << std::endl;  
        }  
    }  
  
    // 队列的大小  
    std::cout << "队列的大小是: " << myQueue.size() << std::endl;  
  
    // 清空队列  
    while (!myQueue.empty()) 
    {  
        myQueue.pop();  
    }

```

---
### priority_queue
std::priority_queue 类是一种容器适配器，他实现经典数据结构中 **堆** 的功能，它提供常数时间的（默认）最大元素查找，对数代价的插入与提取。

优先队列的主要操作发生在容器末端，从末端弹出元素。
std::priority_queue 推入元素后会通过用户提供的 Compare 更改顺序，从末端弹出元素。

为了使 std::queue 能够正常工作，其底层的容器必须满足序列容器的要求和随机访问迭代器。此外，该容器还必须提供以下拥有标准语义的函数：

- `back()` ：类似于 `std::deque::back()`，用于访问末元素。
- `push_back()` ：类似于 `std::deque::push_back()`，用于在末尾添加元素。
- `pop_back()` ：类似于 `std::vector::pop_back()`，用于从末尾移除元素。

>标准容器 std::deque 和 std::vector（包括 std::vector<bool>） 满足这些要求。
>如果在使用 std::queue 时没有特定指定底层容器类型，那么默认情况下会使用 std::deque 作为其底层容器。
因为 std::deque 在大多数情况下的性能表现都是相当优秀的，特别是在元素的添加和移除操作方面。

priority_queue 实现思路和 queue 类模板基本一致，就是调用底层容器接口。只是操作函数的集合改变了。

#### 成员对象
容器适配器作为封装，保有一个 `Container c` 的底层容器作为受保护成员对象。
为了排序，保有一个 `Compare comp` 的比较函数对象，在操作中比较用来保证顺序。
之后的各种操作函数本质上都是调用 `Container c` 的函数。
 
#### 隐式定义的成员函数
- `构造函数`: 初始化容器
- `析构函数`: 销毁容器的每个元素
>销毁 priority_queue 。调用各元素的析构函数，然后解分配所用的存储。
注意，若元素是指针，则不销毁所指向的对象。

- `operator=` : 赋值运算符
> - 复制赋值运算符。以 other 内容的副本替换内容。
    相当于调用 `c = other.c`。
> - 移动赋值运算符。用移动语义以 other 的内容替换内容（即从 other 移动 other 中的数据到此容器中）。
    相当于调用 `c = std::move(other.c)`。

#### 元素访问
- `top` : 访问队首元素
>返回到 priority_queue 中首元素的引用。此元素将是调用 pop() 时第一个移除的元素。
若使用默认比较函数，则返回的元素亦为优先队列中最大的元素。
实际上调用 `c.back()`。


#### 容量
- `empty` : 检查容器适配器是否为空
>检查底层容器是否为空。
实际上调用 `c.empty()`。

- `size` : 返回元素数
>返回底层容器中的元素数。
实际上调用 `c.size()`。

#### 修改器
- `push` : 	插入元素，并对底层容器排序
>将给定的元素 value 推到队尾。
>1) 实际上调用 `c.push_back(value)` ` std::push_heap(c.begin(), c.end(), comp);`。
>2) 实际上调用 `c.push_back(std::move(value))` `std::push_heap(c.begin(), c.end(), comp);`。

- `emplace` : 原位构造元素并排序底层容器
>推入新元素到优先队列。
原位构造元素，即不进行移动或复制操作。以与提供给函数严格相同的实参调用元素的构造函数。
实际上调用 `c.emplace_back(std::forward<Args>(args)...)` `std::push_heap(c.begin(), c.end(), comp)`。

- `pop` : 移除队首元素
>从 priority_queue 移除顶元素。
实际上调用 `c.pop_back()` `std::pop_heap(c.begin(), c.end(), comp)`。

- `swap` : 交换内容
>交换容器适配器与 other 的内容。
实际上调用 `swap(c, other.c)`;

- `std::swap(std::queue)` : 特化 std::swap 算法
>为 std::queue 特化了 std::swap 算法。
交换 lhs 与 rhs 的内容。
实际上调用 `lhs.swap(rhs)`。

```cpp
    // 示例
    // 创建一个包含自定义比较函数的优先队列，实现最小堆  
        auto comp = [](int a, int b) { return a > b; }; // 比较函数降序
        std::priority_queue<int, std::vector<int>, decltype(comp)> pq(comp);  
    // 向优先队列中插入元素  
    pq.push(3);  
    pq.push(1);  
    pq.push(4);  
  
    // 访问队首元素（即优先级最高的元素）  
    int topElement = pq.top();  
    std::cout << "队首元素（最大元素）: " << topElement << std::endl;  
  
    // 检查优先队列是否为空  
    if (!pq.empty()) 
    {  
        std::cout << "优先队列非空。" << std::endl;  
    }  
  
    // 获取优先队列中的元素数量  
    std::cout << "优先队列中的元素数量: " << pq.size() << std::endl;  
  
    // 移除队首元素  
    pq.pop();  
  
    // 再次访问队首元素  
    topElement = pq.top();  
    std::cout << "移除一个元素后，队首元素（最大元素）: " << topElement << std::endl;  
  
    // 使用范围 for循环遍历并打印优先队列中的剩余元素  
    std::cout << "优先队列中的剩余元素: ";  
    while (!pq.empty()) 
    {  
        std::cout << pq.top() << " ";  
        pq.pop();  
    }  
    std::cout << std::endl;  

```

至此，三个容器适配器**栈**，**队列**，**优先队列**，就介绍完了。