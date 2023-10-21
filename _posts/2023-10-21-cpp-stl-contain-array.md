---
title:  【CPP】C++ 标准库（STL）系列教程 序列容器 array
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 序列容器 array 部分。


---
# CPP 序列容器快速入门（1） array

### 引言

序列容器是C++标准模板库（STL）中的重要组成部分，它们允许我们按顺序存储和访问元素。这些容器为开发者提供了灵活且高效的数据结构，用于处理各种复杂的编程任务。

其中，array 作为固定大小的序列容器，为存储固定数量的同类型元素提供了便捷的方式。
通过 array ，我们可以轻松地进行元素的访问、修改和遍历，满足了对静态数据集合的基本操作需求。

在本入门教程中，我们将详细探讨 array 容器的使用方法，帮助读者快速掌握这一基本而实用的工具。


--- 
### array (静态数组)
array 是固定大小的容器，元素严格顺序排列(相当于CPP中的数组的扩展)。

>注意长度为0时 `array.begin() == array.end()` ，此时 `front()` `back()` 是未定义的。

#### 隐式定义的成员函数
- `构造函数`: 遵循聚合初始化的规则初始化 array（注意默认初始化导致非类的 T 的不确定值）
>聚合初始化: 从初始化器列表初始化聚合体。
>- 语法：
>   - T 对象 = { 实参1, 实参2, ... };
>   - T 对象 { 实参1, 实参2, ... };
> - 对于数组，按下标顺序包含所有数组元素
> - 花括号消除：可消除（省略）环绕嵌套的初始化器列表的花括号。
>   - 这种情况下，使用所需数量的初始化器子句初始化对应的子聚合体的各个成员或元素，而后继的各个初始化器子句被用来初始化对象中的后续成员。
>   - 如果对象拥有不带任何成员的子聚合体（空结构体，或只保有静态成员的结构体），那么不允许消除花括号而必须使用一个空的嵌套列表 {}。

- `析构函数`: 销毁 array 的每个元素
- `operator=` : 以来自另一 array 的每个元素重写 array 的对应元素

```cpp
    // 示例

    // 聚合初始化 
    std::array<int, 3> arr1 = {1, 2, 3};
    // 默认初始化，元素值不确定
    std::array<int, 3> arr2;
    
    // 使用赋值操作符  
    arr2 = arr1;
```

#### 元素访问
- `at`: 带越界检查访问指定的元素(引用)
>返回位于指定位置 pos 的元素的引用，有边界检查。
若 pos 不在容器范围内，则抛出 `std::out_of_range` 类型的异常。

- `operator[]`: 访问指定的元素(引用)
>返回位于指定位置 pos 的元素的引用。不进行边界检查。
通过此运算符访问不存在的元素是未定义行为。

- `front`: 访问第一个元素
>返回到容器首元素的引用。
在空容器上对 `front` 的调用是未定义的。

- `back`: 访问最后一个元素。
>返回到容器中最后一个元素的引用
在空容器上对 `back` 的调用是未定义的。

- `data`: 直接访问底层连续存储，返回指向底层元素存储的指针
>对于非空容器，返回的指针与首元素地址比较相等。
如果 `size()` 是 ​0​，那么 `data()` 有可能会也有可能不会返回空指针。

```cpp
    // 示例
    std::array<int, 5> arr = {10, 20, 30, 40, 50};  

    try {  
        std::cout << "Element at index 2: " << arr.at(2) << std::endl; // 访问元素，带边界检查  
    } catch (const std::out_of_range& e) {  
        std::cerr << "Out of range error: " << e.what() << std::endl;  
    }  

    std::cout << "First element: " << arr[0] << std::endl; // 访问第一个元素  
    std::cout << "First element: " << arr.front() << std::endl; // 访问第一个元素  
    std::cout << "Last element: " << arr.back() << std::endl;   // 访问最后一个元素  
    std::cout << "Data pointer: " << static_cast<void*>(arr.data()) << std::endl; // 获取底层指针
```

#### 迭代器
- `begin` `cbegin`: 返回指向起始的迭代器，c 前缀代表 const 不可修改(下同)
>返回指向 array 首元素的迭代器。
如果 array 为空，那么返回的迭代器等于 end()。

- `end` `cend`: 返回指向末尾的迭代器
>返回指向 array 末元素后一元素的迭代器。
此元素表现为占位符；试图访问它导致未定义行为。

- `rbegin` `crbegin`: 返回指向起始的逆向迭代器
>返回指向逆向的 array 的首元素的逆向迭代器。它对应非逆向 array 的末元素。
如果 array 为空，那么返回的迭代器等于 rend()。

- `rend` `crend`: 返回指向末尾的逆向迭代器
>返回指向逆向的 array 末元素后一元素的逆向迭代器。它对应非逆向 array 首元素的前一元素。此元素表现为占位符，试图访问它导致未定义行为。

```cpp
    // 示例
    std::array<int, 5> arr = {10, 20, 30, 40, 50};  
  
    for (std::array<int, 5>::iterator it = arr.begin(); it != arr.end(); ++it) {  
        std::cout << *it << ' '; // 使用迭代器遍历并打印数组元素  
    }  
    std::cout << std::endl;  
  
    // 使用常量迭代器  
    for (std::array<int, 5>::const_iterator it = arr.cbegin(); it != arr.cend(); ++it) {  
        std::cout << *it << ' '; // 遍历并打印数组元素，但无法修改它们  
    }  
    std::cout << std::endl;  
```

#### 容量
- `empty`: 检查容器是否为空
>检查容器是否无元素，begin() == end()
若容器为空则为 true，否则为 false

- `size`: 返回元素数
>返回容器中的元素数，std::distance(begin(), end())

- `max_size`: 返回可容纳的最大元素数
>返回根据系统或库实现限制的容器可保有的元素最大数量，最大容器的 std::distance(begin(), end())

```cpp
    // 示例
    std::array<int, 5> arr;  
    // 检查是否为空
    std::cout << "Is empty? " << std::boolalpha << arr.empty() << std::endl; 
    // 获取元素数量 
    std::cout << "Size: " << arr.size() << std::endl; 
    // 获取最大容量（固定值）  
    std::cout << "Max size: " << arr.max_size() << std::endl;

```

#### 操作
- `fill`: 以指定值填充容器
> void fill( ForwardIt first, ForwardIt last, const T& value );
赋值给定的 value 给 [first, last) 中的各元素。

- `swap`: 交换内容
>将容器内容与 other 的内容交换。
不会导致迭代器和引用关联到别的容器。

```cpp
    // 示例
    std::array<int, 5> arr1;
    // 使用 fill 填充数组  
    std::fill(arr1.begin(), arr1.end(), 100);
    // 使用 swap 交换数组  
    std::array<int, 5> arr2 = {1,2,3,4,5};
    arr1.swap(arr2);
```

至此常见的 array 相关用法已经介绍完毕了，接下来会尝试代码实现这个容器(敬请期待)。