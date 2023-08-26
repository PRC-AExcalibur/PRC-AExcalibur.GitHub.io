---
title:  【CPP】C++ 标准库（STL）系列教程 分配器
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 分配器部分。


---
# C++ 分配器 快速入门
### 引言
在C++标准模板库（STL）中，分配器`Allocator`在内存管理方面扮演着一个至关重要的角色。它为容器提供了分配和释放内存的能力，同时确保了资源管理的高效性和安全性。理解`Allocator`的工作机制对于深入掌握STL容器的内部运作至关重要。

分配器`Allocator`不仅提供了基本的内存分配和释放功能，还允许用户自定义内存管理策略，这对于性能优化和特殊需求的应用场景尤为重要。

本文将介绍解分配器`Allocator`的基本概念、工作原理以及如何在实际编程中利用它。

### 分配器`Allocator`介绍
`Allocator`是一种用于管理容器中对象内存的策略。

在STL中，默认的`Allocator`类型是`std::allocator<T>`，其中`T`是容器存储的对象类型。

它负责为容器中的对象分配和释放内存，同时也提供了构造和析构对象的功能。

`Allocator`的主要职责如下：
1. **内存分配与释放**：`Allocator`可以请求系统分配一块特定大小的连续内存空间，并在不再需要时释放这块内存。
2. **对象构造与析构**：`Allocator`能够调用适当的构造函数来初始化新分配的内存上的对象，并在内存释放前调用析构函数来清理对象状态。
3. **内存块的管理**：对于大型数据集，`Allocator`可以管理多个内存块，以减少内存碎片和提高效率。


### 成员类型
`Allocator`类拥有几种关键的成员类型，它们定义了与类型相关的特性：

-  `value_type` : 定义了`Allocator`管理的对象类型，通常表示为`T`。
-  `size_type` : 是无符号整数类型，用于表示容器的大小。默认是`std::size_t`。
-  `difference_type` : 是有符号整数类型，用于表示容器中元素间的距离。默认是`std::ptrdiff_t`。
-  `propagate_on_container_move_assignment` : 一个类型别名，用于控制在容器移动赋值时是否传播`Allocator`。
  如果`std::true_type`，则会在移动赋值时复制`Allocator`；如果是`std::false_type`，则不会复制。


### 公开成员函数
-  构造函数 : 创建一个新的`Allocator`实例。
>默认情况下，`Allocator`的构造函数是默认构造的，即不接受任何参数。

-  析构函数 : 析构`Allocator`实例。
>由于`Allocator`不持有任何资源，因此其析构函数通常是空的。

-  `address` : 返回给定对象的地址。
>返回 x 的实际地址，即使存在重载的 operator& 也是如此。
>在C++20之前，这是必要的，因为`operator&`可能被重载。从C++20开始，此函数变得可选。

-  `allocate` : 分配足够的内存来存储`n`个`value_type`对象，但不会初始化这些对象。
>调用 ::operator new(std::size_t) 或 ::operator new(std::size_t, std::align_val_t) (C++17 起)分配 n * sizeof(T) 字节的未初始化存储，但何时及如何调用此函数是未指定的。
指针 hint 可用于提供引用的局部性：如果实现支持，那么 allocator 会试图分配尽可能接近 hint 的新内存块。
然后，此函数在该存储中创建一个 T[n] 数组并开始其生存期，但不开始其任何元素的生存期。
如果 T 是不完整类型，那么此函数的使用非良构。

-  `deallocate` : 释放由`allocate`函数分配的内存。
>解分配指针 p 所引用的存储，指针必须是通过先前调用 allocate() 或 allocate_at_least() (C++23 起) 获得的指针。
实参 n 必须等于对原先生成 p 的 allocate() 调用的首个实参，或若 p 由返回 {p, count} 的调用 allocate_at_least(m) 获得，则在范围 [m, count] 中 (C++23 起)；否则行为未定义。
调用 ::operator delete(void*) 或 ::operator delete(void*, std::align_val_t) (C++17 起)，但何时及如何调用是未指定的。

-  `max_size` : 返回`Allocator`能够分配的最大元素数量。
>返回理论上可行的 n 最大值，对于它 allocate(n, 0) 调用可能成功。
大部分实现中，它返回 std::numeric_limits<size_type>::max() / sizeof(value_type)。

-  `construct` : 在分配的内存上构造一个`value_type`对象。
>用全局的布置 new，在 p 指向的未初始化存储中构造 T 类型对象。
>1) 调用 ::new((void*)p) T(val)。
>2) 调用 ::new((void*)p) U(std::forward<Args>(args)...)。

-  `destroy` : 销毁存储在分配的内存中的`value_type`对象。
>调用 p 指向的对象的析构函数。
>1) 调用 p->~T()。
>2) 调用 p->~U()。

### 非成员函数
-  `operator==` `operator!=` : 比较两个`Allocator`实例是否相等/不等。
>比较两个默认分配器。因为默认分配器无状态，故两个默认分配器始终相等。

### 使用`Allocator`
尽管在大多数情况下，使用默认的`std::allocator`就足够了，但在某些场景下，可能需要自定义`Allocator`的行为。

例如，当处理大量数据时，可以通过重写`Allocator`来实现更高效的内存管理策略，比如使用内存池技术。

示例代码如下：
```cpp
#include <vector>
#include <iostream>

#include <memory>

template <typename T>
struct CustomAllocator
{
    using value_type = T;
    using size_type = std::size_t;
    using difference_type = std::ptrdiff_t;
    using propagate_on_container_move_assignment = std::true_type;

    T *allocate(size_type n)
    {
        return static_cast<T *>(::operator new(n * sizeof(T)));
    }

    void deallocate(T *p, size_type)
    {
        ::operator delete(p);
    }

    void construct(T *p, const T &val)
    {
        new (static_cast<void *>(p)) T(val);
    }

    void destroy(T *p)
    {
        p->~T();
    }

    bool operator==(const CustomAllocator &) const noexcept
    {
        return true;
    }

    bool operator!=(const CustomAllocator &) const noexcept
    {
        return false;
    }
};

int main()
{
    std::vector<int, CustomAllocator<int>> vec;
    vec.push_back(666);
    std::cout << "Vector contains: " << vec[0] << std::endl;
    return 0;
}
```

在这个例子中，我们定义了一个简单的`CustomAllocator`。
它重载了`allocate`和`deallocate`等方法，直接使用`new`和`delete`操作符进行内存管理。
然后，我们创建了一个使用自定义分配器的`std::vector`实例并打印。


总的来说，`Allocator`为容器提供了高效、安全的内存管理能力。

通过理解和应用`Allocator`，可以进一步优化代码，特别是在处理大规模数据集或有特殊内存需求的应用中。

至此常见的分配器`Allocator`相关用法已经介绍完毕了。