---
title:  【CPP】C++ 标准库（STL）系列教程 序列容器 deque
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 序列容器 deque 部分。


---
# CPP 序列容器快速入门（3） deque

### 引言

序列容器是C++标准模板库（STL）中的重要组成部分，它们允许我们按顺序存储和访问元素。这些容器为开发者提供了灵活且高效的数据结构，用于处理各种复杂的编程任务。

其中，deque 是一个允许在序列两端快速进行插入和删除操作的容器。deque 提供了与 vector 类似的接口，但与 vector 不同的是，deque 并不是通过连续的内存块来存储元素，而是采用了一种分块存储的策略。这使得deque 在两端操作时的性能优于 vector，特别是当涉及大量元素的移动时。

由于 deque 的分块存储结构，其随机访问（使用索引）的性能可能稍逊于 vector。然而，在需要频繁在两端进行操作的场景中，deque 通常是一个更好的选择。

在本入门教程中，我们将详细探讨 deque 容器的使用方法，帮助读者快速掌握这一基本而实用的工具。


--- 

### deque (双端队列)
deque（double-ended queue，双端队列）是有索引的序列容器，它允许在它的首尾两端快速插入及删除。
>在 deque 任一端的插入或删除不会使指向其余元素的指针或引用失效。
>deque 的元素不是连续存储的：
简单实现采用一系列单独分配的固定尺寸数组存储对象，再用额外的数组存数组的索引；
因此对 deque 的索引访问必须进行二次/多次指针解引用，与之相比 vector 的索引访问只进行一次。

>deque 的存储按需自动扩张及收缩。
>扩张 deque 比扩张 vector 更轻便，因为它不需要将既存元素复制到新内存。
>同时 deque 拥有较大的最小内存开销：只有一个元素的 deque 必须分配一个内部数组。

>deque 上常见操作的复杂度（效率）如下：
>- 随机访问——常数 O(1)。
>- 在结尾或起始插入或移除元素——常数 O(1)。
>- 插入或移除元素——线性 O(n)。

#### 隐式定义的成员函数
- `构造函数`: 初始化容器
- `析构函数`: 销毁容器的每个元素
- `operator=` : 赋值运算符
- `assign` : 将值赋给容器

```cpp
    // 默认构造函数，创建一个空的deque  
    std::deque<int> emptyDeque;  
  
    // 使用迭代器范围构造deque  
    std::vector<int> vec = {1, 2, 3, 4, 5};  
    std::deque<int> deque_from_vec(vec.begin(), vec.end());  
  
    // 使用指定数量的元素和初值构造deque  
    std::deque<int> ten_zeros(10, 0);  
  
    // 拷贝构造函数  
    std::deque<int> copy_deque(dequeFromVec);  
  
    // 初始化列表构造函数  
    std::deque<int> init_list_deque = {6, 7, 8, 9, 10};  

    std::deque<int> deque1 = {1, 2, 3};  
    std::deque<int> deque2;  
    // 使用赋值运算符将deque1的内容赋给deque2  
    deque2 = deque1;  

    // 使用assign函数将新内容赋给deque2  
    deque2.assign(5, 4); 

    // 使用assign函数通过迭代器范围赋值  
    deque2.assign(deque1.begin(), deque1.end()); 

```

#### 元素访问
对于元素访问，deque vector 和 array 的接口是一致的。

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
    std::deque<int, 5> arr = {10, 20, 30, 40, 50};  

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
对于迭代器接口，deque vector 和 array 的接口是一致的。
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
    std::deque<int> arr = {10, 20, 30, 40, 50};  
  
    for (std::deque<int>::iterator it = arr.begin(); it != arr.end(); ++it) {  
        std::cout << *it << ' '; // 使用迭代器遍历并打印数组元素  
    }  
    std::cout << std::endl;  
  
    // 使用常量迭代器  
    for (std::deque<int>::const_iterator it = arr.cbegin(); it != arr.cend(); ++it) {  
        std::cout << *it << ' '; // 遍历并打印数组元素，但无法修改它们  
    }  
    std::cout << std::endl;  
```

#### 容量
deque 的容量接口中有一部分与 array 是一致的：
- `empty`: 检查容器是否为空
- `size`: 返回元素数
- `max_size`: 返回可容纳的最大元素数

由于 deque 不是完全连续存储，每次额外申请一个内部数组大小，因此也不需要程序员特地申请特定大小的内存了。不过 std 还是提供了释放未使用内存的接口：
- `shrink_to_fit`: 释放未使用的内存（减少内存的使用）
>请求移除未使用的容量，是减少使用内存而不更改序列的大小的非强制性请求。请求是否达成依赖于实现。
所有迭代器和到元素的引用失效。

```cpp
    // 示例
    std::deque<int> d;  
  
    // empty: 检查容器是否为空   
    std::cout << std::boolalpha << "Is deque empty? " << d.empty() << std::endl; // 输出: Is deque empty? true  
  
    // 添加一些元素  
    d.push_back(1);
    d.push_back(2);
    d.push_back(3);
  
    // size: 返回元素数  
    std::cout << "Size of deque: " << d.size() << std::endl; // 输出: Size of deque: 3  
  
    // max_size: 返回可容纳的最大元素数  
    std::cout << "Max size of deque: " << d.max_size() << std::endl;  
  
    // shrink_to_fit: 尝试释放未使用的内存  
    d.shrink_to_fit();
```


---
#### 修改器

deque 支持前后添加元素，而 vector 支持在容器尾部添加元素，vector 操作是 deque 的子集。
因此 deque 支持 vector 所有的修改器接口。

- `clear` : 清除内容
>从容器擦除所有元素。此调用后 size() 返回零。

>使任何指代所含元素的引用、指针和迭代器失效。 任何尾后迭代器也会失效。

- `insert` : 插入元素
>插入元素到容器中的指定位置。
>1. 在 pos 前插入 value；
返回指向被插入 value 的迭代器。
>2. 在 pos 前插入 value 的 count 个副本；
返回指向首个被插入元素的迭代器， count == 0 时返回 pos。
>3. 在 pos 前插入来自迭代器范围 [first, last) 的元素；
返回指向首个被插入元素的迭代器， first == last 时返回 pos。
>4. 在 pos 前插入来自 initializer_list ilist 的元素

>所有迭代器（包括 end() 迭代器）都会失效。
引用也会失效，除非 pos == begin() 或 pos == end()，此时它们不会失效。

- `emplace` : 原位构造元素
>在紧接 pos 之前的位置向容器插入新元素；返回指向被安置的元素的迭代器。

>迭代器失效原理与 insert 相同。

- `erase` : 擦除元素
>从容器擦除指定的元素。
>1. 移除位于 pos 的元素；
返回最后移除元素之后的迭代器，如果 pos 指代末元素，那么返回 end() 迭代器。
>2. 移除范围 [first, last) 中的元素；
返回最后移除元素之后的迭代器，如果在移除前 last == end()，那么返回更新的 end() 迭代器。如果范围 [first, last) 为空，那么返回 last。

>擦除中间元素，所有迭代器和引用都会失效；
被擦除的元素在容器的开头或末尾时，只有指向被擦除元素的迭代器或引用会失效。
除非被擦除元素在容器开头，而尾元素未被擦除，否则 end() 迭代器失效。

- `push_back` : 将元素添加到容器末尾
>追加给定元素 value 到容器尾。
>1. 对于常量引用，初始化新元素为 value 的副本。
>2. 对于右值引用，移动 value 进新元素。

>所有迭代器（包括 end() 迭代器）都会失效。引用不会失效。

- `emplace_back` : 在容器末尾原位构造元素
>添加新元素到容器尾。元素通过 std::allocator_traits::construct 构造，通常用放置式 new 于容器所提供的位置原位构造元素。
>迭代器失效原理与 push_back 相同。

- `pop_back` 移除末元素
>移除容器的末元素。
在空容器上调用 pop_back 是未定义行为。
指向被擦除元素的迭代器和引用会失效。end() 迭代器也会失效。
其他引用和迭代器不受影响。

- `resize` : 改变存储元素的个数
>重设容器大小以容纳 count 个元素。
如果当前大小大于 count，那么减小容器到它的前 count 个元素。
如果当前大小小于 count，追加额外的默认插入的元素/ value 的副本。

- `swap` : 交换内容
>将内容以及容量与 other 的交换。不在单独的元素上调用任何移动、复制或交换操作。
所有迭代器和引用仍然有效。end() 迭代器失效。

>注意与 array 的区别：array 会交换所有的元素； deque 的实质是交换指针。

```cpp
    // 示例
    std::deque<int> deq = {1, 2, 3, 4, 5};  
  
    // clear  
    deq.clear();  
    deq = {1, 2, 3, 4, 5};  

    // insert 单个元素  
    deq.insert(deq.begin() + 2, 100); // 在索引2的位置插入100  

    // insert 多个相同元素  
    deq.insert(deq.begin() + 4, 2, 200); // 在索引4的位置插入2个200  
    // insert 范围 [first, last)  
    std::vector<int> temp = {300, 400, 500};  
    deq.insert(deq.end(), temp.begin(), temp.end()); // 在末尾插入temp的所有元素
    // 使用列表初始化插入  
    deq.insert(deq.begin(), {600, 700}); // 在开头插入两个元素  

    // emplace  
    deq.emplace(deq.begin() + 3, 200); // 在索引3的位置原位构造200   
  
    // erase 单个元素  
    deq.erase(deq.begin() + 2); // 移除索引2的元素（即元素3） 
    // erase 范围 [first, last)  
    deq.erase(deq.begin() + 1, deq.begin() + 4); // 移除索引1到3（不包含4的元素（即元素2, 4, 5）  
  
    // push_back  
    deq.push_back(300); // 在末尾添加300  
  
    // emplace_back  
    deq.emplace_back(400); // 在末尾原位构造400  
  
    // pop_back  
    deq.pop_back(); // 移除末尾元素  
  
    // resize  
    deq.resize(15, 500); // 调整大小为15，新元素用500填充  

    // swap  
    std::deque<int> deq2 = {9, 8, 7, 6, 5, 4, 3, 2, 1};  
    deq.swap(deq2); // 交换deq和deq2的内容  
```

同时， deque 支持在容器首部添加/删除元素：
- `push_front` : 插入元素到容器起始
>前附给定元素 value 到容器起始。
所有迭代器（包括 end() 迭代器）都会失效。没有引用会失效。

- `emplace_front` : 在容器头部原位构造元素
>插入新元素到容器开头。通过 std::allocator_traits::construct 构造元素，通常用放置式 new 在容器所提供的位置原位构造元素。
所有迭代器（包括 end() 迭代器）都会失效。没有引用会失效。

- `pop_front` : 移除首元素
>移除容器首元素。若容器中无元素，则行为未定义。
指向被擦除元素的迭代器和引用会失效。
如果元素是容器的最后元素，那么 end() 迭代器也会失效。

```cpp
    // 示例
    std::deque<int> deq = {1, 2, 3, 4, 5};  
  
    // push_front  
    deq.push_front(300); // 在队首添加300  
  
    // emplace_front  
    deq.emplace_front(400); // 在队首原位构造400  
  
    // pop_front  
    deq.pop_front(); // 移除队首元素   
```

至此常见的 deque 相关用法已经介绍完毕了，接下来会尝试代码实现这个容器(敬请期待)。