---
title:  【CPP】C++ 标准库（STL）系列教程 序列容器 vector
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 序列容器 vector 部分。


---
# CPP 序列容器快速入门（2） vector

### 引言

序列容器是C++标准模板库（STL）中的重要组成部分，它们允许我们按顺序存储和访问元素。这些容器为开发者提供了灵活且高效的数据结构，用于处理各种复杂的编程任务。

其中，vector 作为可以动态增长的序列容器，能够根据需要自动分配和释放内存。它允许我们像操作普通数组一样，通过索引快速访问元素，同时还提供了在尾部高效添加和删除元素的方法。这使得 vector 成为处理可变大小数据集的理想选择。

在本入门教程中，我们将详细探讨 vector 容器的使用方法，帮助读者快速掌握这一基本而实用的工具。

---

### vector (动态数组)
vector 是可变大小的容器，元素严格顺序排列(相当于CPP中的动态数组的扩展)。
>vector 的存储是自动管理的，按需扩张收缩。
vector 通常占用多于静态数组的空间，因为要分配更多内存以管理将来的增长。

>vector扩容的步骤如下：
>1. 申请更大的内存空间
>2. 将旧空间的数据按顺序复制/移动到新空间
>3. 释放旧空间
>
>因此扩容是比较耗时的操作，应该尽量减少扩容次数。
>我们再介绍一下 vector 的扩容策略。
> >vector 只在额外内存耗尽时进行重分配，一般新分配的空间是当前空间的 1.5~2 倍。
分配的内存总量可用 capacity() 函数查询。
可以通过调用 shrink_to_fit()返回多余的内存给系统。
>
>>如果元素数量已知，那么 reserve() 函数可用于消除重分配。
但是调用 reserve() 后，插入只会在它将导致 vector 的大小大于 capacity() 的值时触发重新分配。

>vector 上的常见操作复杂度（效率）如下：
>- 随机访问——常数 O(1)。
>- 在末尾插入或移除元素——均摊常数 O(1)。
>- 插入或移除元素——与到 vector 结尾的距离成线性 O(n)。


#### 隐式定义的成员函数
- `构造函数`: 初始化容器
- `析构函数`: 销毁容器的每个元素
- `operator=` : 赋值运算符
  - 复制赋值运算符。以 other 内容的副本替换内容。
  - 移动赋值运算符。用移动语义以 other 的内容替换内容（即从 other 移动 other 中的数据到此容器中）。之后 other 处于合法但未指定的状态。
- `assign` : 将值赋给容器

```cpp
    // 示例
    // 使用默认构造函数创建一个空的 vector
    std::vector<int> vec1;
    
    // 使用初始化列表构造一个包含几个元素的 vector
    std::vector<int> vec2 = {1, 2, 3, 4, 5};
    
    // 使用拷贝构造函数创建一个新 vector，它是 vec2 的拷贝
    std::vector<int> vec3(vec2);
    
    // 使用赋值操作符将 vec2 的内容赋给 vec1
    vec1 = vec2;

    // 使用 assign 替换所有元素
    vec1.assign(3, 7); // 赋值为 3 个 7
```


#### 元素访问
对于元素访问，vector 和 array 的接口是一致的。
因此在工程实践中，大多数情况都会用 vector 代替 array , 因为做相同的事时，vector 比 array 更灵活。

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
    std::vector<int> arr = {10, 20, 30, 40, 50};  

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
vector 的迭代器接口与 array 也是完全一致的。
- `begin` `cbegin`: 返回指向起始的迭代器，c 前缀代表 const 不可修改(下同)
>返回指向 容器 首元素的迭代器。
如果 容器 为空，那么返回的迭代器等于 end()。

- `end` `cend`: 返回指向末尾的迭代器
>返回指向 容器 末元素后一元素的迭代器。
此元素表现为占位符；试图访问它导致未定义行为。

- `rbegin` `crbegin`: 返回指向起始的逆向迭代器
>返回指向逆向的 容器 的首元素的逆向迭代器。它对应非逆向 容器 的末元素。
如果 容器 为空，那么返回的迭代器等于 rend()。

- `rend` `crend`: 返回指向末尾的逆向迭代器
>返回指向逆向的 容器 末元素后一元素的逆向迭代器。它对应非逆向 容器 首元素的前一元素。此元素表现为占位符，试图访问它导致未定义行为。

```cpp
    // 示例
    std::vector<int> arr = {10, 20, 30, 40, 50};  
  
    for (std::vector<int>::iterator it = arr.begin(); it != arr.end(); ++it) {  
        std::cout << *it << ' '; // 使用迭代器遍历并打印数组元素  
    }  
    std::cout << std::endl;  
  
    // 使用常量迭代器  
    for (std::vector<int>::const_iterator it = arr.cbegin(); it != arr.cend(); ++it) {  
        std::cout << *it << ' '; // 遍历并打印数组元素，但无法修改它们  
    }  
    std::cout << std::endl;  
```


#### 容量
vector 的容量接口中有一部分与 array 是一致的：
- `empty`: 检查容器是否为空
- `size`: 返回元素数
- `max_size`: 返回可容纳的最大元素数

但由于vector是动态数组，因此可以改变容量：
- `reserve`: 预留存储空间
>增加 vector 的容量（不重新分配存储的情况下能最多能持有的元素的数量）到大于或等于 new_cap 的值。
reserve() 不会更改 vector 的 size()。

>只有 new_cap 大于当前的 capacity() 时才会分配新存储;
若分配新存储， 则所有迭代器和到元素的引用失效。实际是因为旧空间已经释放了。
调用 reserve() 后，插入只会在它将导致 vector 的大小大于 capacity() 的值时触发重新分配。

>注意 reserve 不能减少分配内存，需要用 shrink_to_fit ,且该操作依赖具体实现。

- `capacity`: 返回当前存储空间能够容纳的元素数
>返回当前分配存储的容量（元素数）。

- `shrink_to_fit`: 释放未使用的内存（减少内存的使用）
>请求移除未使用的容量，是减少 capacity() 到 size()非强制性请求。请求是否达成依赖于实现。
若分配新存储， 则所有迭代器和到元素的引用失效。

```cpp
    // 示例
    std::vector<int> vec = {1,2,3};

    // 检查是否为空
    std::cout << "Is empty? " << std::boolalpha << vec.empty() << std::endl;
    
    // 获取元素数量
    std::cout << "Size: " << vec.size() << std::endl;
    
    // 获取当前存储空间能够容纳的元素数
    std::cout << "Capacity: " << vec.capacity() << std::endl;
    
    // 预留更多存储空间
    vec.reserve(10);
    std::cout << "Reserved capacity: " << vec.capacity() << std::endl;
    
    // 尝试减少内存使用
    vec.shrink_to_fit();
    std::cout << "After shrink_to_fit: " << vec.capacity() << std::endl;
```


#### 修改器
- `clear` : 清除内容
>从容器擦除所有元素。此调用后 size() 返回零。

>使任何指代所含元素的引用、指针和迭代器失效。 任何尾后迭代器也会失效。
保持 vector 的 capacity() 不变。

- `insert` : 插入元素
>插入元素到容器中的指定位置。
>1. 在 pos 前插入 value；
返回指向被插入 value 的迭代器。
>2. 在 pos 前插入 value 的 count 个副本；
返回指向首个被插入元素的迭代器， count == 0 时返回 pos。
>3. 在 pos 前插入来自迭代器范围 [first, last) 的元素；
返回指向首个被插入元素的迭代器， first == last 时返回 pos。
>4. 在 pos 前插入来自 initializer_list ilist 的元素

>如果操作后新的 size() 大于原 capacity() 则会发生重分配，指代元素的所有迭代器（包括 end() 迭代器）和所有引用均会失效。
若无重分配，则仅插入点之前的迭代器和引用保持有效。

- `emplace` : 原位构造元素
>在紧接 pos 之前的位置向容器插入新元素；返回指向被安置的元素的迭代器。

>迭代器失效原理与 insert 相同。
- `erase` : 擦除元素
>从容器擦除指定的元素。
>1. 移除位于 pos 的元素；
返回最后移除元素之后的迭代器，如果 pos 指代末元素，那么返回 end() 迭代器。
>2. 移除范围 [first, last) 中的元素；
返回最后移除元素之后的迭代器，如果在移除前 last == end()，那么返回更新的 end() 迭代器。如果范围 [first, last) 为空，那么返回 last。

>擦除后，指向位于擦除点或其后的元素的迭代器（包括 end() 迭代器）和引用均会失效。

- `push_back` : 将元素添加到容器末尾
>追加给定元素 value 到容器尾。
>1. 对于常量引用，初始化新元素为 value 的副本。
>2. 对于右值引用，移动 value 进新元素。

>迭代器失效原理与 insert 相同。

- `emplace_back` : 在容器末尾原位构造元素
>添加新元素到容器尾。元素通过 std::allocator_traits::construct 构造，通常用放置式 new 于容器所提供的位置原位构造元素。
>迭代器失效原理与 insert 相同。

- `pop_back` 移除末元素
>移除容器的末元素。
在空容器上调用 pop_back 是未定义行为。
指向最后元素的迭代器（包括 end() 迭代器）和引用失效。

- `resize` : 改变存储元素的个数
>重设容器大小以容纳 count 个元素。
如果当前大小大于 count，那么减小容器到它的前 count 个元素。
如果当前大小小于 count，追加额外的默认插入的元素/ value 的副本。

- `swap` : 交换内容
>将内容以及容量与 other 的交换。不在单独的元素上调用任何移动、复制或交换操作。
所有迭代器和引用仍然有效。end() 迭代器失效。

>注意与 array 的区别：array 会交换所有的元素； vector 的实质是交换指针。


```cpp
    // 示例
    std::vector<int> vec = {1, 2, 3, 4, 5};  
  
    // clear  
    vec.clear();  
    vec = {1, 2, 3, 4, 5};  

    // insert 单个元素  
    vec.insert(vec.begin() + 2, 100); // 在索引2的位置插入100  

    // insert 多个相同元素  
    vec.insert(vec.begin() + 4, 2, 200); // 在索引4的位置插入2个200  
    // insert 范围 [first, last)  
    std::vector<int> temp = {300, 400, 500};  
    vec.insert(vec.end(), temp.begin(), temp.end()); // 在末尾插入temp的所有元素
    // 使用列表初始化插入  
    vec.insert(vec.begin(), {600, 700}); // 在开头插入两个元素  

    // emplace  
    vec.emplace(vec.begin() + 3, 200); // 在索引3的位置原位构造200   
  
    // erase 单个元素  
    vec.erase(vec.begin() + 2); // 移除索引2的元素（即元素3） 
    // erase 范围 [first, last)  
    vec.erase(vec.begin() + 1, vec.begin() + 4); // 移除索引1到3（不包含4的元素（即元素2, 4, 5）  
  
    // push_back  
    vec.push_back(300); // 在末尾添加300  
  
    // emplace_back  
    vec.emplace_back(400); // 在末尾原位构造400  
  
    // pop_back  
    vec.pop_back(); // 移除末尾元素  
  
    // resize  
    vec.resize(15, 500); // 调整大小为15，新元素用500填充  

    // swap  
    std::vector<int> vec2 = {9, 8, 7, 6, 5, 4, 3, 2, 1};  
    vec.swap(vec2); // 交换vec和vec2的内容  
```

至此常见的 vector 相关用法已经介绍完毕了，接下来会尝试代码实现这个容器(敬请期待)。