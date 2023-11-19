---
title:  【CPP】C++ 标准库（STL）系列教程 序列容器 list/forward_list
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 序列容器 list 与 forward_list 部分。


---
# CPP 序列容器快速入门（4） list 与 forward_list

### 引言

序列容器是C++标准模板库（STL）中的重要组成部分，它们允许我们按顺序存储和访问元素。这些容器为开发者提供了灵活且高效的数据结构，用于处理各种复杂的编程任务。

其中，list 允许在序列的任何位置快速进行插入和删除操作。与 vector 和 deque 不同，list 并不是通过连续的内存块来存储元素，而是采用链式存储结构。这种结构使得 list 在插入和删除元素时无需移动其他元素，因此效率更高。然而，由于链式存储的特性，std::list 的随机访问性能相对较差，不如 vector 和 deque.

在本入门教程中，我们将详细探讨 list 容器的使用方法，帮助读者快速掌握这一基本而实用的工具。


--- 

### list

list 是一个容器，它支持在容器的任何位置进行常数时间的元素插入和移除操作。这个容器并不支持快速的随机访问，通常实现为双向链表。与 forward_list 相比，list 提供了双向迭代能力，但在空间效率上稍逊一筹。

在 list 内部或在不同 list 对象之间添加、移除和移动元素时，并不会导致已存在的迭代器或引用失效。
迭代器只有在对应元素被显式删除时才会变得无效。

这种特性使得 list 在需要对元素进行频繁插入和删除操作的场景中非常有用，因为它能够保持操作的效率，同时保持对容器内其他元素的稳定引用。
>list 上的常见操作复杂度（效率）如下：
>- 随机访问——常数——与到 list 首尾的距离成线性 O(n)。
>- 插入或移除元素——均摊常数 O(1)。

#### 隐式定义的成员函数
- `构造函数`: 初始化容器
- `析构函数`: 销毁容器的每个元素
- `operator=` : 赋值运算符
- `assign` : 将值赋给容器

```cpp
    // 默认构造函数，创建一个空 list  
    std::list<int> emptyList;  
      
    // 使用初始值列表构造函数  
    std::list<int> listWithElements = {1, 2, 3, 4, 5};  
      
    // 拷贝构造函数  
    std::list<int> copyOfList(listWithElements);   

    std::list<int> list1 = {1, 2, 3};  
    std::list<int> list2;  
      
    // 使用赋值运算符  
    list2 = list1;  

    // 使用assign函数将新内容赋给list2  
    list2.assign(5, 4); 

    // 使用assign函数通过迭代器范围赋值  
    list2.assign(list1.begin(), list1.end()); 
```

#### 元素访问
对于元素访问，list 和 deque vector array 的接口差距很大。
由于不支持随机访问，因此没有接口 `at` `operator[]` , 只有 `front` 和 `back`。
那我们怎么访问中间元素呢？我们可以使用迭代器获取中间元素，并对迭代器解引用获取对应的值。

- `front`: 访问第一个元素
>返回到容器首元素的引用。
在空容器上对 `front` 的调用是未定义的。

- `back`: 访问最后一个元素。
>返回到容器中最后一个元素的引用
在空容器上对 `back` 的调用是未定义的。

```cpp
    // 示例
    std::list<int> larr = {10, 20, 30, 40, 50}; 
    std::cout << "First element: " << larr.front() << std::endl; // 访问首元素  
    std::cout << "Last element: " << larr.back() << std::endl; // 访问尾元素  
```

#### 迭代器
对于迭代器接口，list 和 deque vector array 的接口是一致的。
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
    std::list<int> arr = {10, 20, 30, 40, 50};  
  
    for (std::list<int>::iterator it = arr.begin(); it != arr.end(); ++it) {  
        std::cout << *it << ' '; // 使用迭代器遍历并打印list元素  
    }  
    std::cout << std::endl;  
  
    // 使用常量迭代器  
    for (std::list<int>::const_iterator it = arr.cbegin(); it != arr.cend(); ++it) {  
        std::cout << *it << ' '; // 遍历并打印list元素，但无法修改它们  
    }  
    std::cout << std::endl;  
```

#### 容量
list 的容量接口与 array , deque 是一致的：
- `empty`: 检查容器是否为空
- `size`: 返回元素数
- `max_size`: 返回可容纳的最大元素数

由于 list 每次添加都会新申请空间，没有预分配的空间浪费，因此没有 `shrink_to_fit`。


```cpp
    // 示例
    std::list<int> d;  
  
    // empty: 检查容器是否为空   
    std::cout << std::boolalpha << "Is list empty? " << d.empty() << std::endl; // 输出: Is list empty? true  
  
    // 添加一些元素  
    d.push_back(1);
    d.push_back(2);
    d.push_back(3);
  
    // size: 返回元素数  
    std::cout << "Size of list: " << d.size() << std::endl; // 输出: Size of deque: 3  
  
    // max_size: 返回可容纳的最大元素数  
    std::cout << "Max size of list: " << d.max_size() << std::endl;  
```


---
#### 修改器
list 的修改器接口与 deque 是一致的：

- `clear` : 清除内容
>从容器擦除所有元素。此调用后 size() 返回零。

>使任何指代所含元素的引用、指针和迭代器失效。 任何尾后迭代器保持有效。

- `insert` : 插入元素
>插入元素到容器中的指定位置。
>1. 在 pos 前插入 value；
返回指向被插入 value 的迭代器。
>2. 在 pos 前插入 value 的 count 个副本；
返回指向首个被插入元素的迭代器， count == 0 时返回 pos。
>3. 在 pos 前插入来自迭代器范围 [first, last) 的元素；
返回指向首个被插入元素的迭代器， first == last 时返回 pos。
>4. 在 pos 前插入来自 initializer_list ilist 的元素

>没有引用和迭代器会失效。

- `emplace` : 原位构造元素
>在紧接 pos 之前的位置向容器插入新元素；返回指向被安置的元素的迭代器。

>迭代器失效原理与 insert 相同。

- `erase` : 擦除元素
>从容器擦除指定的元素。
>1. 移除位于 pos 的元素；
返回最后移除元素之后的迭代器，如果 pos 指代末元素，那么返回 end() 迭代器。
>2. 移除范围 [first, last) 中的元素；
返回最后移除元素之后的迭代器，如果在移除前 last == end()，那么返回更新的 end() 迭代器。如果范围 [first, last) 为空，那么返回 last。

>擦除中间元素，指向被擦除元素的迭代器和引用会失效。其他引用和迭代器不受影响。

- `push_back` : 将元素添加到容器末尾
>追加给定元素 value 到容器尾。
>1. 对于常量引用，初始化新元素为 value 的副本。
>2. 对于右值引用，移动 value 进新元素。

>没有引用和迭代器会失效。

- `emplace_back` : 在容器末尾原位构造元素
>添加新元素到容器尾。元素通过 std::allocator_traits::construct 构造，通常用放置式 new 于容器所提供的位置原位构造元素。
>迭代器失效原理与 push_back 相同。

- `pop_back` 移除末元素
>移除容器的末元素。
在空容器上调用 pop_back 是未定义行为。
指向被擦除元素的迭代器和引用会失效。

- `push_front` : 插入元素到容器起始
>前附给定元素 value 到容器起始。
没有引用和迭代器会失效。

- `emplace_front` : 在容器头部原位构造元素
>插入新元素到容器开头。通过 std::allocator_traits::construct 构造元素，通常用放置式 new 在容器所提供的位置原位构造元素。
没有引用和迭代器会失效。

- `pop_front` : 移除首元素
>移除容器首元素。若容器中无元素，则行为未定义。
指向被擦除元素的迭代器和引用会失效。

- `resize` : 改变存储元素的个数
>重设容器大小以容纳 count 个元素。
如果当前大小大于 count，那么减小容器到它的前 count 个元素。
如果当前大小小于 count，追加额外的默认插入的元素/ value 的副本。

- `swap` : 交换内容
>将内容以及容量与 other 的交换。不在单独的元素上调用任何移动、复制或交换操作。
所有迭代器和引用仍然有效。在操作后，未指明保有此容器中 end() 值的迭代器指代此容器还是另一容器。

>注意与 array 的区别：array 会交换所有的元素； list 的实质是交换指针。

```cpp
    // 示例
    std::list<int> list1 = {1, 2, 3, 4, 5};  
  
    // clear  
    list1.clear();  
    list1 = {1, 2, 3, 4, 5};  

    // insert 单个元素  
    std::list<int>::iterator it = list1.begin();
    it++;
    it++;
    list1.insert(it, 100); // 在索引2的位置插入100   

    // insert 多个相同元素  
    it++;
    list1.insert(it , 2, 200); // 在索引4的位置插入2个200  

    // insert 范围 [first, last)  
    std::vector<int> temp = {300, 400, 500};  
    list1.insert(list1.end(), temp.begin(), temp.end()); // 在末尾插入temp的所有元素

    // 使用列表初始化插入  
    list1.insert(list1.begin(), {600, 700}); // 在开头插入两个元素  

    // emplace  
    it = list1.begin();
    it++;
    it++;
    it++;
    list1.emplace(it, 200); // 在索引3的位置原位构造200    
  
    // erase 单个元素  
    it--;
    list1.erase((it)); // 移除索引2的元素（即元素3） 

    // erase 范围 [first, last)  
    it = list1.begin();
    it++;
    auto it2 = it;
    it2++;
    it2++;
    list1.erase(it, it2); // 移除索引1到3（不包含4的元素（即元素2, 4, 5）
  
    // push_back  
    list1.push_back(300); // 在末尾添加300  
  
    // emplace_back  
    list1.emplace_back(400); // 在末尾原位构造400  
  
    // pop_back  
    list1.pop_back(); // 移除末尾元素  

    // push_front  
    list1.push_front(300); // 在首添加300  
  
    // emplace_front  
    list1.emplace_front(400); // 在首原位构造400  
  
    // pop_front  
    list1.pop_front(); // 移除首元素  

    // resize  
    list1.resize(15, 500); // 调整大小为15，新元素用500填充  

    // swap  
    std::list<int> list2 = {9, 8, 7, 6, 5, 4, 3, 2, 1};  
    list1.swap(list2); // 交换list1和list2的内容  
```

#### 操作
- `merge` : 合并两个有序列表
>如果 other 与 *this 指代同一对象，那么什么也不做。
否则，将 other 合并到 *this。两个链表都应有序。用 operator< 或 comp 比较元素。
不复制元素，并且在操作后容器 other 会变为空。此操作是稳定的：对于两个链表中的等价元素，来自 *this 的元素始终在来自 other 的元素之前，并且不更改 *this 和 other 的等价元素顺序。
迭代器和引用不会失效。指向或指代从 other 移走的元素的指针、引用和迭代器将不再涉及 other，而是会指代 *this 中的相同元素。


- `splice` : 从另一个 list 中移动元素
>从一个 list 转移元素给另一个。
不复制或移动元素，仅对链表结点的内部指针进行重指向。
没有迭代器或引用会失效，指向被移动元素的迭代器保持有效，但现在指代到 *this 中，而不是到 other 中。
>
>- 从 other 转移所有元素到 *this 中。元素被插入到 pos 指向的元素之前。操作后容器 other 变为空。
>- 从 other 转移 it 指向的元素到 *this。元素被插入到 pos 指向的元素之前。
>- 从 other 转移范围 [first, last) 中的元素到 *this。元素被插入到 pos 指向的元素之前。


- `remove` `remove_if` : 
>remove：移除所有等于 value 的元素。用 operator== 或 二元谓词 p 比较元素
remove_if：移除所有满足特定标准的元素。
只有指向被移除元素的迭代器和引用会失效。


- `reverse` : 反转元素的顺序
>逆转容器中的元素顺序。迭代器和引用不会失效。

- `unique` : 删除连续的重复元素
>从容器移除所有相继的的重复元素。用 operator== 或 二元谓词 p 比较元素，只留下相等元素组中的第一个元素。
只有到被移除元素的迭代器和引用会失效。
如果对应的的比较器没有建立等价关系，那么行为未定义。

- `sort` : 对元素进行排序
>排序元素，并保持等价元素的顺序。不会导致迭代器和引用失效。
用 operator< 或 comp 比较元素。因此默认升序


```cpp
    std::list<int> list1 = {1, 3, 5, 7};  
    std::list<int> list2 = {2, 4, 6, 8};  
      
    // merge 
    list1.merge(list2); // 假设 list1 和 list2 是有序的


    // splice
    list1 = {1, 2, 3};  
    list2 = {4, 5, 6};  
    auto it = list1.begin();  
    std::advance(it, 2); // 移动到 list1 的第 3 个位置  
    list1.splice(it, list2); // 将 list2 的所有元素移动到 list1 的 it 位置之前 

    list1 = {1, 2, 3, 7, 8, 9};  
    list2 = {4, 5, 6, 10, 11, 12};  
    it = list1.begin();  
    std::advance(it, 3); // 移动到 list1 的第 4 个位置  
    // 转移 list2 中从第 2 个到第 4 个元素（不包含第 4 个）到 list1 的 it 位置之前  
    auto it2 = std::next(list2.begin(), 1); // list2 的第 2 个元素  
    auto it3 = std::next(list2.begin(), 3); // list2 的第 4 个元素（不包含）  
    list1.splice(it, list2, it2, it3);  


    std::list<int> list{8, 7, 5, 9, 0, 1, 3, 2, 6, 4,4,4，11，1};
    // remove
    list.remove(1);
    list.remove_if([](int n){ return n > 10; });

    // reverse
    list.reverse();

    // unique
    list.unique();

    // sort
    list.sort();//升序
    list.sort(std::greater<int>());//降序

```

至此常见的 list 相关用法已经介绍完毕了，接下来会尝试代码实现这个容器(敬请期待)。

### forward_list
forward_list 是 list 的简化版，通常实现是单链表。
它的接口和 list 基本一致，只是少了需要双指针才能实现的接口，在此只进行简要介绍，主要说明 forward_list 与 list 的区别，建议通过 list 来理解forward_list.
>因为少了指向上一个节点的指针，容器其实保存的是头节点。

>对于迭代器：
迭代器中所有和 rbegin 类似的反向迭代器接口都不支持。

>对于函数：
函数中所有和 back 相关的接口都不支持。

对于基础的元素修改接口，`insert` `emplace` `erase` 分别对应 `insert_after` `emplace_after` `erase_after`.

其他的和 list 理解上就没什么区别了。
